<template>
    <article class="task_container">

        <button type="button" class="btn_empty" @click="taskClear">清空</button>

        <h1 class="title1">待辦事項</h1>

        <div class="task_add_block" ref="taskAddBlock">
          <!-- focus事件 : 打字欄位輸入焦點顯示 blur : 不顯示 -->
            <input type="text" 
            class="task_name" 
            placeholder="輸入待辦事項…"
            @focus="toggleShadow"
            @blur="toggleShadow"
            v-model="taskText" 
            @keyup.enter="taskAdd">    <!--輸入完文字按enter即可更新-->

            <button type="button" 
            class="task_add"
            @click="taskAdd">新增</button>
        </div>

        <div class="task_list_parent">

            <ul class="task_list" ref="taskList">
             <TaskItem :tasks="tasks" 
             @task-edit="taskEdit"
             @task-star="taskStar"
             @task-swap="taskSwap"
             @task-remove="taskRemove"></TaskItem> <!-- 等於後的tasks是陣列資料 -->
            </ul>
        </div>
    </article>
</template>

<script>
  import TaskItem from './TaskItem.vue';


  export default {
    components: {TaskItem},
    data(){
      return{
        taskText: "" , // 回傳使用者打字的內容
        tasks: [
          // {
          //   id: "a",
          //   name: "一",
          //   star: 0,
          //   editable: false
          // }
        ]
      };
    },
    beforeMount(){
      // console.log("ttt");
      let tasks = JSON.parse(localStorage.getItem("tasks"));
      if(tasks){
        this.tasks = tasks;
      }
    },
    methods: {
      toggleShadow(){
        this.$refs.taskAddBlock.classList.toggle("-on");
        // console.log(this.$refs.taskAddBlock);
      },
      taskAdd(){
        if(this.taskText != ""){   // 如果沒打字按新增 不會有反應
          // alert("yyyy");
          this.tasks.unshift({
            id: Date.now(),
            name: this.taskText,
            star: 0,
            editable: false
          });
          this.taskText = "";
          localStorage.setItem("tasks", JSON.stringify(this.tasks));
        }
      },
      taskClear(){
        let r = confirm("確認清空?");
        if(r){
          for(let i = 0; i< this.$refs.taskList.children.length; i++){
            this.$refs.taskList.children[i].classList.add("fade_out");

            // 先淡出再將資料清空
            setTimeout(()=> {
              this.tasks = [];
              localStorage.clear();
              // localStorage.removeItem("tasks"); //刪除特定li
            },1000);

          }
        }
      },
      taskEdit(e, i){
        if(this.tasks[i].editable){
          if(this.tasks[i].name == ""){
            alert("請輸入資料");
          }else{
            this.tasks[i].editable =  !this.tasks[i].editable;
            localStorage.setItem("tasks", JSON.stringify(this.tasks));
          }
        }else{
          this.tasks[i].editable =  !this.tasks[i].editable;
        }
      },
      taskStar(e, i, star){
        // alert(star);
        this.tasks[i].star = star;  
        localStorage.setItem("tasks", JSON.stringify(this.tasks)); //將資料暫存
      },
      taskSwap(e, i, direction){
        // alert(direction);
        if(direction == "up" && i != 0){
          // alert("up");
          [this.tasks[i-1], this.tasks[i]] = [this.tasks[i], this.tasks[i-1]];
          localStorage.setItem("tasks", JSON.stringify(this.tasks)); //將資料暫存

        }

        if(direction == "down" && i != (this.tasks.length - 1)){ //陣列長度
          // alert("down");
          [this.tasks[i], this.tasks[i+1]] = [this.tasks[i+1], this.tasks[i]];
          localStorage.setItem("tasks", JSON.stringify(this.tasks)); //將資料暫存

        }
      },
      taskRemove(e, i){
        // console.log(i);
        let r = confirm("確定要移除嗎?");
        if(r){
          e.target.closest("li").classList.add("fade_out");
          setTimeout(()=>{
            this.tasks.splice(i, 1); // 從選取的索引值刪掉一個資料
            localStorage.setItem("tasks", JSON.stringify(this.tasks)); //將資料暫存
          },1000);
        }
      }
    }
  }
</script>

<style lang="sass" scoped>
  article.task_container
    width: 600px
    margin: 0 auto
    border-radius: 5px
    padding: 10px
    box-shadow: 1px 1px 5px gray
    max-width: 100%
    h1.title1
      font-size: 2.4rem
      margin: 0 0 10px 0
  
    /* ===== 任務新增 ===== */
    div.task_add_block
      font-size: 0
      transition: transform .5s, box-shadow .5s
      &.-on
        box-shadow: 0 0 5px gray
        transform: scale(1.01)
        transform-origin: center center
  
      input.task_name
        width: calc(100% - 50px)
        border: 1px solid lightgray
        border-radius: 3px 0 0 3px
        height: 40px
        font-size: 2rem
        padding: 5px 10px
        outline: none
        display: inline-block
        vertical-align: top
    input.task_name::placeholder
      color: lightgray
  
    div
      &.task_add_block button.task_add
        display: inline-block
        width: 50px
        height: 40px
        vertical-align: top
        border: 1px solid lightgray
        border-left: 0
        background-color: white
        box-shadow: none
        font-size: 1.6rem
        cursor: pointer
        outline: none
        border-radius: 0 3px 3px 0
  
        &:active
          box-shadow: 1px 1px 3px lightgray inset, -1px -1px 3px lightgray inset
          background-color: #fcfcfc
          font-size: 1.5rem
      &.task_list_parent
        margin-top: 30px
        ul.task_list
          margin: 0
          padding: 0
          list-style: none
  
    /* ===== 清空按鈕 ===== */
    button.btn_empty
      float: right
      background: none
      background-color: white
      padding: 0
      margin: 0
      border: 1px solid lightgray
      border-radius: 3px
      height: 30px
      width: 50px
      outline: none
      font-size: 1.6rem
      cursor: pointer

/* 一大片 li 的 css */
li
  margin-bottom: 20px
  border-bottom: 1px solid gray
  padding-bottom: 20px
  opacity: 1
  transition: opacity 1s

  &.fade_out
    opacity: 0

  &:first-child
    border-top: 1px solid black
    padding-top: 20px

    button.btn_up
      background-color: lightgray !important
      cursor: not-allowed !important
      color: gray

  &:last-child
    margin-bottom: 0
    border-bottom: 0

    button.btn_down
      background-color: lightgray !important
      cursor: not-allowed !important
      color: gray

  > div.item_flex
    display: flex

    /* border: 1px solid black; */

    > div
      /* border: 1px solid blue; */

      &.left_block
        width: 100px
        flex-shrink: 0

        div.btn_flex
          display: flex

          > button
            flex-grow: 1
            outline: none
            height: 40px
            font-size: 1.6rem
            padding: 0
            margin: 0
            background: none
            background-color: white
            border: 0
            cursor: pointer
            border: 1px solid #eee

      &.middle_block
        font-size: 1.8rem
        flex-grow: 1
        padding-left: 10px
        padding-right: 10px

        p.para
          margin: 0
        /* ===== 重要性的星號 ===== */
        div.star_block
          display: inline-block

          > span.star
            cursor: pointer
            display: inline-block
            margin-right: 3px

            &.-on
              color: yellow
        
        input.task_name_update
          width: 100%
          border: 1px solid lightgray
          border-radius: 3px 0 0 3px
          height: 40px
          font-size: 2rem
          padding: 5px 10px
          outline: none

          &::placeholder
            color: lightgray

      &.right_block
        width: 100px
        flex-shrink: 0

        div.btn_flex
          display: flex

          > button
            flex-grow: 1
            outline: none
            height: 40px
            font-size: 1.6rem
            padding: 0
            margin: 0
            background: none
            background-color: white
            border: 0
            cursor: pointer
            border: 1px solid #eee
</style>