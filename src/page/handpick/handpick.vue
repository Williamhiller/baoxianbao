<template>
    <div class="handpick-page" v-if="specialData != null">
      <img :src="specialData.imgUrl" class="top-img">
      <div class="top-card">
        <div class="cont">
          <h2>{{specialData.specialTitle}}</h2>
          <p>{{specialData.summary}}</p>
        </div>
      </div>

      <div class="block block1" v-for="(template,index) in templates">
        <div class="padding" v-if="template != null">
          <h2 class="sub_title">
            <span>0{{template.specialTempletId}}</span> {{template.specialTitle}}
          </h2>
        </div>
        <div class="card_box">
          <div class="card">
            <router-link v-for="item in template.articles"
                         tag="div" :to="{path:'/article',query:{articleId:item.articleId}}"
                         class="item item-avatar avatar-left">
              <div class="avatar"><img src="../../assets/image/inner.png" alt=""></div>
              <div class="item-body">
                <h2>{{item.title}}</h2>
                <p>{{item.summary}}</p>
              </div>
            </router-link>

            <div class="more" @click="getMoreArticle(index)" v-show="!item.isLast">
              <img src="../../assets/image/arrow_down.png" alt="">
            </div>
          </div>
        </div>
      </div>

      <!--<div class="block block2" v-if="templates[1] != null">-->
        <!--<div class="padding">-->
          <!--<h2 class="sub_title">-->
            <!--<span>02</span> 投保时的事宜-->
          <!--</h2>-->
        <!--</div>-->
        <!--<div class="card_box">-->
          <!--<div class="card">-->

            <!--<div class="item item-avatar avatar-left" v-for="item in list1">-->
              <!--<div class="avatar round"><img src="../../assets/image/inner.png" alt=""></div>-->
              <!--<div class="item-body">-->
                <!--<h2>健康告知</h2>-->
                <!--<p>一二三四五六</p>-->
              <!--</div>-->
            <!--</div>-->

          <!--</div>-->
        <!--</div>-->
      <!--</div>-->

      <!--<div class="block block3" v-if="templates[2] != null">-->
        <!--<div class="padding">-->
          <!--<h2 class="sub_title">-->
            <!--<span>03</span> 理赔-->
          <!--</h2>-->
        <!--</div>-->
        <!--<div class="card_box">-->
          <!--<div class="card">-->
            <!--<div><img src="../../assets/image/inner.png" alt=""></div>-->
            <!--<p>我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要</p>-->

            <!--<div class="text-right see"><span class="a">点击查看全文</span></div>-->
          <!--</div>-->
        <!--</div>-->
      <!--</div>-->

      <!--<div class="block block4" v-if="templates[3] != null">-->
        <!--<div class="padding">-->
          <!--<h2 class="sub_title">-->
            <!--<span>04</span> 多图文滑块-->
          <!--</h2>-->
          <!--<p class="digest">我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要我是摘要</p>-->
        <!--</div>-->
        <!--<div class="swiper-container">-->
          <!--<div class="swiper-wrapper">-->
            <!--<div class="swiper-slide" v-for="item in 6">-->
              <!--<div class="img_box">-->
                <!--<img src="../../assets/image/slide.png" alt="">-->
              <!--</div>-->
              <!--<div class="tit">我是标题</div>-->
            <!--</div>-->
          <!--</div>-->

        <!--</div>-->
      <!--</div>-->

      <tabs active="sign"></tabs>
    </div>
</template>

<script>
  import getData from '../../service/getData'
  import tabs from '../../components/tabs/tabs'
  import Swiper from 'swiper'

  export default {
    name: "handpick",
    data () {
      return {
        list1: [1,2,3],
        specialData : null,
        templates : [null,null,null,null]
      }
    },
    components:{
      tabs
    },
    mounted () {
      new Swiper('.swiper-container', {
        slidesPerView : 2,
        spaceBetween : 0,
        freeMode: true,
        observer : true, //修改swiper自己或子元素时，自动初始化swiper
        observeParents : true//修改swiper的父元素时，自动初始化swiper
      });

    },
    methods: {
      getSpecial() {
        getData.getSpecial(1).then((specialRes)=>{
          this.specialData = specialRes.data

          let promiseArr = []
          specialRes.data.templets.forEach((item,index)=>{
            promiseArr.push(getData.getSpecialArticleList({
              specialTempletId: item.specialTempletId,
              pageNumber : 1,
              pageSize : 5
            }))
          })

          Promise.all(promiseArr).then((res)=> {
            res.data.forEach((item,index)=>{
              this.templates[index] = item.data || {}
              this.templates[index].pageNumber = 1
            })
          })
        })
      },
      /**
       * 获取更多文章
       * @param index
       */
      getMoreArticle (index) {
        let item = this.templates[index]
        let pageNumber = item.pageNumber
        getData.getAuthorArticleList({
          specialTempletId: item.specialTempletId,
          pageNumber : pageNumber +1,
          pageSize : 5
        }).then((list)=>{
          item.pageNumber = pageNumber + 1
          item.articles.push(...list.data.articles)
          if(item.articles.length >= item.total) {
            item.isLast = true
          }

          this.$set(this.templates,index,item)
        })
      },
    }
  }
</script>

<style scoped lang="scss">
  @import "../../style/variable";
  .handpick-page {
    overflow-x: hidden;
    /*padding-top: 3rem;*/
    background-color: #fff;
    /*background-image: url("../../assets/image/inner.png");*/
    /*background-repeat: no-repeat;*/
    /*background-size: 100% 4rem;*/

    padding-bottom: $tabs-height + 0.2rem;
  }
  .top-img {
    width: 100%;
    height: 4rem;
  }

  .top-card {
    padding: 0 .48rem;
    margin-top: -1rem;
    .cont {
      background-color: #fff;
      border-radius: 0.12rem;
      padding: .28rem .2rem;
      height: 2rem;
      box-shadow: 0 .04rem .26rem 0 rgba(#000,0.06);
      h2 {
        color: $base-color;
        font-size: .36rem;
        font-weight: 600;
        line-height: 1.6;
      }
      p {
        font-size: .24rem;
        color: #666;
        padding: .1rem .16rem;
        line-height: 1.3;
      }
    }
  }

  .padding {
    padding: .36rem;
  }
  .sub_title {
    font-weight: bold;
    font-size: .46rem;
    color: #333;
    padding-bottom: .24rem;
    border-bottom: .01rem solid $border-color;
    span {
      font-size: .86rem;
      color: $base-color;
      margin-right: .1rem;
    }
  }
  .card_box {
    padding: 0 .24rem;

    .card {
      border-radius: .16rem;
      padding: .1rem .2rem;
      box-shadow: 0 .04rem .26rem 0 rgba(#C3C3C3,0.5);

      .more {
        text-align: center;
        padding: .24rem;
        img {
          width: .5rem;
          height: auto;
        }
      }
    }

  }
  .block1 {
    .item {
      border-bottom: 0.01rem solid $border-color;
    }
  }
  .block2 {
    .card {
      overflow: hidden;
    }
    .item {
      width: 50%;
      float: left;
      margin-bottom: .2rem;

      h2 {
        padding-top: .1rem;
      }
    }
  }

  .block3 {
    .card {
      padding: .2rem;
      img {
        width: 100%;
        height: 2.4rem;
      }
      p {
        color: #666666;
        font-size: .24rem;
        line-height: 1.4;
        padding: .1rem 0;
      }

      .see {
        padding: .2rem 0;
      }
    }
  }
  .block4 {
    .digest {
      color: #666666;
      font-size: .24rem;
      line-height: 1.4;
      padding: .1rem 0;
    }
  }

  .swiper-container {
    width: 6.4rem;
    margin-left: .16rem;
    overflow: visible;
    .swiper-slide {
      padding-left: .24rem;
      .img_box {
        width: 100%;
        img {
          width: 100%;
          height: 2.8rem;
          box-shadow: 0 .02rem .08rem 0 rgba(#9F9F9F,0.5);
          border-radius: .16rem;
        }
      }
      .tit {
        font-size: .28rem;
        color: $base-color;
        font-weight: bold;
        line-height: 2;
      }

    }
  }
</style>
