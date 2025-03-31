<script lang="ts" setup>
  import { getPosts, Post } from '@/api/api';
  import { onBeforeMount, ref } from 'vue';

  const posts = ref<Post[]>();
  let displayedPosts = ref<Post[]>();
  let page=0;
  const pageSize=3;
  let pageCount=0;


  onBeforeMount(() => {
      loadPosts().then(()=>{
        //we calculate page count based on data and create buttons
        const count = posts.value.length;
        pageCount = Math.ceil(count/pageSize);
        fillButtonRow();
        displayedPosts.value = posts.value.slice(0, pageSize);
        });  
    });

  async function loadPosts() {
    posts.value = (await getPosts()).posts;
  }

  /**this function assembles the button row*/
  function fillButtonRow(){
    const btnRow = document.querySelector(".js-page-switch-row");
    //check for existence
    if(btnRow){
      const shown =[];
      //we select which pages to show for selection
      for(let i=0;i<pageCount;i++){
        if((i<=1) || (i>=pageCount-2) || (Math.abs(i-page)<=2)){
          shown.push(i);
        }
      }
      //we construct the inner html
      let inner="";
      for(let i=0;i<shown.length; i++){
        if(i>=1 && shown[i]-shown[i-1]>1){
          inner+="<div> ...</div>";
        }
        const isActive = (shown[i]==page) ? "active" : "";
        inner+=`
          <button class="page-button ${isActive}" data-page="${shown[i]}">
          ${shown[i]+1}
          </button>
        `;
      }
      //it gets displayed
      btnRow.innerHTML=inner;
      //we add event listeners to buttons:
      const btns = document.querySelectorAll(".page-button");
      btns.forEach(button => {
        button.addEventListener("click",()=>{
          //we change the page
          const newPage = button.dataset.page;
          page=newPage;
          //we rerun this function
          fillButtonRow();
          displayedPosts.value = posts.value.slice(page*pageSize,page*pageSize+pageSize);
          //console.log(`start: ${page*pageSize} end: ${(page)*pageSize+pageSize} page: ${page} pageSize: ${pageSize}`)
        });
      });
    } else {
      console.error("no button row was found =(");
    }
  }
</script>

<template>
  <h1>Posts list</h1>
  <div class="js-page-switch-row">
  </div>
  <RouterLink
    class="post"
    v-for="(post, index) of displayedPosts"
    :to="{ name: 'post', params: { postId: post.id } }"
    :key="index"
  >
    <div class="post__id">#{{ post.id }}</div>
    <div class="post__title">{{ post.title }}</div>
    <div>{{ post.body }}</div>
  </RouterLink>
</template>

<style scoped>
  .post {
    display: grid;
    grid-template-columns: min-content 1fr;
    grid-template-rows: repeat(2, auto);
    gap: 8px;

    & + & {
      margin-top: 12px;
      padding-top: 12px;
      border-top: 1px solid #ccc;
    }

    &__id {
      grid-row: -1/1;
    }

    &__title {
      font-weight: 700;
    }
  }

  .js-page-switch-row{
    font-size: 18px;
    display: flex;
    flex-direction: row;
    justify-items: flex-start;
  }

  .js-page-switch-row:deep(.page-button){
    font-size: 18px;
    border-width: 0;
    background-color: white;
    cursor: pointer;
  }

  .js-page-switch-row:deep(.active){
    color: red;
  }
</style>
