<style>
    .comment-offset{
        margin-left: 10px;
    }
    .comment-offset2{
        margin-left: 30px;
    }
    .comment-btn-offset{
        margin-bottom: 10px;
    }
    .time1{
        font-size: small;
        position:absolute;
        right:25px;
        bottom:12px;
    }
    .time2{
        font-size: small;
        position:absolute;
        right:10px;
        bottom: 0;
    }
    .reply{
        font-size: small;
        color: #95a5a6;
    }
</style>
<template>
    <div id="commentlist">
        <h4>吐槽列表：</h4>
        <ul id="comment_list" class="list-group">
            <h5 v-if="{{comments.length == 0}}">暂无吐糟</h5>

            <li v-for="comment in comments" class="list-group-item" style="padding-left: 6px;padding-right: 6px;">
                <div class="container-fluid" style="padding-left: 0;padding-right: 0;">
                    <div class="row">
                        <div class="col-xs-2" style="padding: 0;">
                            <center class="comment-offset2">
                                <span style="word-break: break-all;font-size: 14px;"> {{comment.nickname}} </span><br>
                            </center>
                        </div>
                        <div class="col-xs-10">
                            <p class="comment-offset"> {{comment.content}} </p><br>
                            <div v-if="comment.show_reply_form" >
                                <reply_form :comment="comment"></reply_form>
                            </div>
                            <button class="btn btn-primary btn-xs comment-offset comment-btn-offset" v-else v-on:click="show_reply_form(comment)">回复</button>
                            <time class="time1"> {{new Date(comment.time).Format("yyyy-MM-dd  hh:mm:ss")}} </time>
                            <br/>
                        </div>
                    </div>
                    <ul class="list-group">
                        <li v-for="reply in comment.replys" class="list-group-item">
                            <div class="row">
                                <div class="col-xs-2" style="padding-right:0;margin: 0;">
                                    <center style="word-break: break-all;">
                                        <span style="font-size: 14px;"> {{reply.nickname}}&nbsp;
                                            <span class="reply">回复</span>&nbsp;
                                            {{reply.replyer_name}}
                                        </span><br>
                                    </center>

                                </div>
                                <div class="col-xs-10">
                                    <p> {{reply.content}} </p>
                                    <div v-if="reply.show_reply_form" >
                                        <reply_form :comment="reply"></reply_form>
                                    </div>
                                    <button class="btn btn-primary btn-xs" v-else v-on:click="show_reply_form(reply)">回复</button>
                                    <time class="time2"> {{new Date(reply.time).Format("yyyy-MM-dd  hh:mm:ss")}} </time>
                                    <br/>
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </li>
        </ul>
    </div>
</template>
<script type="text/ecmascript-6">
    import request from 'superagent/lib/client'
    import ReplyFrom from './ReplyForm.vue'

    export default {
        components:{
            reply_form: ReplyFrom
        },
        data(){
            return{
                comments:[]
            }
        },
        created () {
            let that = this;
            request.get('api/comment_list')
                    .set('Accept', 'application/json')
                    .end(function(err, res){
                        if (err == null){
                            that.parse_comments(JSON.parse(res.text).data);
                            that.comments.reverse()
                        }
                    });
        },
        methods:{
            no_reply_form(){
                for (var i = 0; i < this.comments.length; i++) {
                    var comment = this.comments[i];
                    comment.show_reply_form = false;
                    for (var j = 0; j < comment.replys.length; j++){
                        var reply = comment.replys[j]
                        reply.show_reply_form = false;
                    }
                }

            },
            show_reply_form(comment){
                this.no_reply_form();
                comment.show_reply_form = ~comment.show_reply_form;
            },
            parse_comments(list){//解析评论
                this.comments = [];
                var comment_list = this.get_comments_by_parent_id(list, 0);
                for (var i = 0; i < comment_list.length; i++) {
                    var comment = comment_list[i];
                    comment.show_reply_form = false;
                    comment.replys = this.parse_replys(list, comment.id);
                    this.comments.push(comment);
                }
            },
            get_comments_by_parent_id(list, parent_id){//获取制定parent_id的comments
                var comments =[];
                for (var i = 0; i < list.length; i++) {
                    var comment = list[i];
                    if (comment.parent_id == parent_id) {
                        comments.push(comment);
                    }
                }
                return comments;
            },
            parse_replys(list, parent_id){//获取某一个评论的所有回复
                let that = this;
                var comment_replys = [];
                function parse_replys_by_parent_id(list, parent_id){
                    var replys = that.get_comments_by_parent_id(list, parent_id);
                    for (var i = 0; i < replys.length; i++ ) {
                        var reply = replys[i];
                        reply.show_reply_form = false;
                        //递归添加回复，如果下面还有回复的话就添加
                        comment_replys.push(reply);
                        parse_replys_by_parent_id(list, reply.id);
                    }
                }
                parse_replys_by_parent_id(list, parent_id);
                return comment_replys;
            }
        }
    };
</script>