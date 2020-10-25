---
layout: post
title: thymeleaf + ajax实现页面局部刷新
categories: Blog
description: ajax实现页面局部刷新
keywords: thymeleaf ajax 局部刷新
---
项目使用的是springboot+thymeleaf+ajax+mybatis。在实现评论回复功能的时候遇到了一个难题：如何实现文章详情页面中评论功能的局部刷新。
开始的时候一切顺利，后台实体、服务层、控制层将数据完成封装并传递给前端页面（thymeleaf）。
####评论实体
```java
package com.jinghuan.tron.web.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.jinghuan.common.util.DateUtil;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import org.springframework.format.annotation.DateTimeFormat;

import javax.persistence.Column;
import javax.persistence.Table;
import java.util.Date;

@Getter
@Setter
@ToString
@Table(name = "`bl_comment`")
public class BL_Comment {
    private Integer b2_id;
    private Integer b2_pid;
    private Integer b0_id;
    private String b0_name;
    private Integer b1_id;
    private String b2_content;
    @Column(name = "`create_time`")
    @DateTimeFormat(pattern = DateUtil.DEFAULT_FORMAT_PATTERN_DATETIME)
    @JsonFormat(pattern = DateUtil.DEFAULT_FORMAT_PATTERN_DATETIME, timezone = DateUtil.DEFAULT_TIME_ZONE_TYPE)
    private Date create_time;

    @Column(name = "`modify_time`")
    //@DateTimeFormat(pattern = DateUtil.DEFAULT_FORMAT_PATTERN_DATETIME)
    //@JsonFormat(pattern = DateUtil.DEFAULT_FORMAT_PATTERN_DATETIME, timezone = DateUtil.DEFAULT_TIME_ZONE_TYPE)
    private Date modify_time;

}
```
####服务层
```java
@Service
public class BL_CommentServiceImpl implements BL_CommentService {

    @Autowired
    private BL_CommentMapper bl_commentMapper;
    //存放迭代找出的所有子代的集合
    private List<BL_CommentVO> tempReplys = new ArrayList<>();

    /**
     * @Description: 查询评论
     * @Param:
     * @Return: 评论消息
     */
    @Override
    public List<BL_CommentVO> queryComments(BL_CommentVO queryCommentVO) {
        //查询出父节点
        List<BL_CommentVO> comments = bl_commentMapper.findCommentByParentIdIsNull(queryCommentVO);
        for(BL_CommentVO comment : comments){
            Integer b2_id = comment.getB2_id();
            String parentNickname1 = comment.getB0_name();
            queryCommentVO.setB2_id(b2_id);
            List<BL_CommentVO> childComments = bl_commentMapper.findCommentByParentIdIsNotNull(queryCommentVO);
            //查询出子评论
            combineChildren(childComments, parentNickname1);
            comment.setSubCommentVOs(tempReplys);
            tempReplys = new ArrayList<>();
        }
        return comments;
    }

    /**
     * @Description: 查询出子评论
     * @Param: childComments：所有子评论
     * @Param: parentNickname1：父评论的姓名
     * @Return:
     */
    private void combineChildren(List<BL_CommentVO> childComments, String parentNickname1) {
        //判断是否有一级子回复
        if(childComments.size() > 0){
            //循环找出子评论的id
            for(BL_CommentVO childComment : childComments){
                String parentNickname = childComment.getB0_name();
                childComment.setB2_pName(parentNickname1);
                tempReplys.add(childComment);
                Integer childId = childComment.getB2_id();
                //查询二级以及所有子集回复
                recursively(childId, parentNickname);
            }
        }
    }

    /**
     * @Description: 循环迭代找出子集回复
     * @Param: childId：子评论的id
     * @Param: parentNickname1：子评论的姓名
     * @Return:
     */
    private void recursively(Integer childId, String parentNickname1) {
        //根据子一级评论的id找到子二级评论
        List<BL_CommentVO> replayComments = bl_commentMapper.findByReplayId(childId);

        if(replayComments.size() > 0){
            for(BL_CommentVO replayComment : replayComments){
                String parentNickname = replayComment.getB0_name();
                replayComment.setB2_pName(parentNickname1);
                Integer replayId = replayComment.getB0_id();
                tempReplys.add(replayComment);
                //循环迭代找出子集回复
                recursively(replayId,parentNickname);
            }
        }
    }

    @Override
    //存储评论信息
    public int insertNewComment(BL_Comment comment) {
        return bl_commentMapper.insertNewComment(comment);
    }
}
```
####控制层（Controller）
```java
@Controller
@RequestMapping(value = "/")
public class DispatchController {

    ...

    /**
     * 添加新评论
     * @param model
     * @param articleId
     * @param content
     * @return
     */
    @Transactional
    @RequestMapping(value = "/newComment", method = RequestMethod.POST)
    public String insertNewComment(Model model,
                                                         @RequestParam(value = "articleId") String articleId,
                                                         @RequestParam(value = "content") String content){
        if(articleId == null || articleId.length() <= 0 || content == null || content.length() <= 0) return null;
        // 插入评论
        BL_Comment bl_comment = new BL_Comment();
        bl_comment.setB1_id(Integer.parseInt(articleId));
        bl_comment.setB2_content(content);
        bl_comment.setB0_id(0);
        bl_comment.setB0_name("Crucio");
        bl_commentService.insertNewComment(bl_comment);
        // 获取新添加的评论id
        Integer newCommentId = bl_comment.getB2_id();
        if(newCommentId == null || newCommentId <=0) return null;
        //查询当前文章下的评论信息
        BL_CommentVO queryCommentVO = new BL_CommentVO();
        queryCommentVO.setB1_id(Integer.parseInt(articleId));
        List<BL_CommentVO> comments = bl_commentService.queryComments(queryCommentVO);
        model.addAttribute("comments", comments);
        return "blog_post::comments";
    }

    ...
}
```
####前端评论部分
```html
<!-- /Comments -->
  <div  id="comment_container" class="comments" th:if="!${#lists.isEmpty(comments)}" th:fragment="comment_container">
   <h4>Comments</h4>
   <!--<div class="add_comment c_after"><a class="btn_m" href="#">Add Comment</a></div>-->
   <!--<div th:include="this::row(${comments})"/>-->
   <ul th:each="comment,index:${comments}">
    <li th:if="${#lists.isEmpty(comment.subCommentVOs)}">
     <div class="info"><img src="images/avatar.png" alt="" /><strong th:text="${comment.b0_name}">Jason Smith</strong>  |  <i>15th February 2012</i>  |  <a href="#">Reply</a></div>
     <div th:utext="${comment.b2_content}"></div>
    </li>
    <!-- 子集评论 -->
    <li th:if="not ${#lists.isEmpty(comment.subCommentVOs)}" >
     <div class="info"><img src="images/avatar.png" alt="" /><strong th:text="${comment.b0_name}">Jason Smith</strong>  |  <i>15th February 2012</i>  |  <a href="#">Reply</a></div>
     <div th:utext="${comment.b2_content}"></div>
     <ul>
      <li th:each="reply,index : ${comment.subCommentVOs}">
       <div class="info"><img src="images/avatar.png" alt="" /><strong th:text="${reply.b0_name}">Jason Smith</strong>  |  <i>15th February 2012</i>  |  <a href="#">Reply</a></div>
       <div><a th:text="|@ ${reply.b2_pName}||"></a><span  th:text="${reply.b2_content}"></span></div>
      </li>
     </ul>
    </li>
   </ul>
  </div>
  <!-- /Comments -->
```
####添加评论
```html
   <!-- /Leave a Comment -->
   <div class="leave_comment">
    <h4>Leave a Comment</h4>
    <form method="post" onsubmit="return false;" action="">
     <p><label for="namet">Name</label>(required)<br /><input id="namet" type="text" /></p>
     <p><label for="mailt">E-mail</label>(required)<br /><input id="mailt" type="text" /></p>
     <p><label for="website">Website</label><br /><input id="website" type="text" /></p>
     <p><label for="message">Message</label>(required)<br /><textarea id="message"></textarea></p>
     <p><input id="post-new-comment" class="btn_m" type="submit" value="Add Comment" /></p>
    </form>
   </div>
```
####新增评论并局部刷新
```javascript
   <script type="application/javascript">
       // 新增评论
       $("#post-new-comment").on("click", function postComment() {
           //获取文章id
           var articleId = $("#blog_id").val();
           //获取评论内容
           var content = $("#message").val();
           if(articleId === null || Trim(articleId).length <= 0){
               alert("添加评论失败，请刷新页面重试！");
               return false;
           }
           if(content == null || Trim(content).length <= 0){
               alert("请输入评论内容！");
               return false;
           }
           // 解除提交按钮点击事件绑定
           // $("#post-new-comment").unbind("click");
           //发出POST请求
           $.ajax({
               url:"/newComment",
               data: {articleId:articleId,content:content},
               type: "POST",
               dataType: 'text',
               success: function (data) {
                   $("#comment_container").html(data);
               },
               error:function (e) {
                   alert("提交评论失败，请稍后重试！");
               }
           });
           // $.post("../newComment",{articleId:articleId,content:content}, function(data){
           //     $("#comment_container").load(data);
           // });
       });
       /**
        * 过滤字符串去掉标签和空格
        * @param str
        * @returns {string}
        * @constructor
        */
       function Trim(str) {
           var newStr = str.replace(/<[^>]+>/g, "");//去掉所有的html标记
           return (newStr.replace(/&nbsp;/g, "")).trim(); //去掉所有的&nbsp;
       }
   </script>
   <!-- /Leave a Comment -->
```
注意上面控制类中的返回值"blog_post::comments"，在网上查了很多文章，都是这么写的，但在我的项目中却不起作用，新增完评论后始终无法局部刷新页面。取而代之总是显示“blog_post::comments”这个字符在页面上。又看到有人使用Jquery的load方法，而不是html方法。我也跟着尝试了一下，结果居然刷新了，但同时在id为“comment_container”的div中也多了很多东西，F12一看原来load方法将整个blog_post（文章详情）页面重新在这个div中加载了一遍，汗！多种方法尝试终失败，我把注意力转向