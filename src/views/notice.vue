<template>
  <div class="main">
    <!-- 公告 -->
    <div class="notice-main bra">
      <div ref='notice' class="notice-content">
        <div class="notice flex" v-for="(item, index) in noticeOldArr" :key='index' @click='noticeEvent'>
          <text class="c255 f35">公告</text>
          <div class="notice-lines"></div>
          <text class="lines1 notice-text f28 fw4">{{item}}</text>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  const animation = weex.requireModule('animation')
  export default {
    data() {
      return {
        noticeOldArr: []
      }
    },
    mounted() {
      let index = 1,
       noticeItem = this.$refs.notice,
       noticeArr = ['1号公司正式放假4天，请大家知悉！111111111111', '2号公司正式放假4天，请大家知悉！111111111111',
        '3 号公司正式放假4天，请大家知悉！111111111111', '4号公司正式放假4天，请大家知悉！111111111111 '
      ],
      newFirstItem = noticeArr[0],
      newLastItem = noticeArr[noticeArr.length - 1];
      noticeArr.unshift(newLastItem)
      noticeArr.push(newFirstItem)
      this.noticeOldArr = noticeArr
      let noticeArrLength = this.noticeOldArr.length
      setInterval(() => {
        index++
        if (noticeArrLength - 1 == index) {
          index = 1
          this.animationNotice(noticeItem, 0, 0)
        }
        this.animationNotice(noticeItem, index, 800)
      }, 3000);
    },
    methods: {
      // 更多
      noticeEvent() {
        this.$alert()
      },
      animationNotice(noticeItem, index, duration) {
        animation.transition(noticeItem, {
          styles: {
            transform: 'translateY(-' + 88 * index + 'px)',
          },
          duration: duration,
          timingFunction: 'ease',
          needLayout: false,
          delay: 0
        }, function () {})
      }
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
  .main {
    flex: 1;
    padding-top: 100px;
    background-color: #666;
  }

  .notice-main {
    overflow: hidden;
  }

  .notice {
    height: 88px;
    background-color: #fff;
    padding: 0 24px;
  }

  .notice-lines {
    width: 1px;
    height: 44px;
    margin: 0 20px;
    background: rgba(216, 216, 216, 1);
  }

  .notice-text {
    width: 600px;
    color: rgba(77, 76, 79, 1);
  }

  .notice-content {
    height: 88px;
    background-color: #fff;
    transform: translateY(-88px);
  }
</style>