<template>
    <main class="main">
        <div class="detail-wrap" ref="detailWrapRef">
            <div class="detail" ref="detailRef">
                <note
                        v-if="$store.state.details[$route.query.id]"
                        ref="noteRef"
                        :des="$store.state.details[$route.query.id].description"
                        :author="$store.state.details[$route.query.id].owner.login"
                        :avatar="$store.state.details[$route.query.id].owner.avatar_url"
                        :time="$util.getDateDiff($store.state.details[$route.query.id].created_at)"
                        :detail="$store.state.details[$route.query.id].files ? $store.state.details[$route.query.id].files.blog.content : ''"
                ></note>
            </div>
        </div>
    </main>
</template>

<script>
  import Note from '../components/Note'

  export default {
    name: 'detail',
    components: {
      'note': Note,
    },
    data() {
      return {
        blog: {},
        loading: false
      }
    },
    methods: {
      getBlog() {
        this.$tips('正在加载', 'loading');
        this.$ajax.get(`https://api.github.com/gists/${this.$route.query.id}?time=${new Date().getTime()}`).then((res) => {
          this.$store.commit('SET_DETAILS', {
            key: this.$route.query.id,
            value: res.data
          })
          this.$tips('加载成功', 'correct', 2000);
          this.$nextTick(function () {
            this.$refs.detailRef.scrollTop = this.$store.state.scrollTops[this.$route.query.id]
          })
        }).catch(()=>{
          this.$tips('加载失败', 'wrong', 2000);
        })
      }
    },
    activated() {
      if(!this.$store.state.details[this.$route.query.id] || !this.$store.state.details[this.$route.query.id].files) {
        this.getBlog()
      }
    },
    //记住滚动位置
    beforeRouteLeave(to, from, next) {
      this.$store.commit('SET_SCROLLTOPS', {
        key: this.$route.query.id,
        value: this.$refs.detailWrapRef.scrollTop
      });
      next();
    },
    beforeRouteEnter(to, from, next) {
      next(vm => {
        vm.$refs.detailWrapRef.scrollTop = vm.$store.state.scrollTops[vm.$route.query.id]
      })
    }
  }
</script>


<style lang="scss" scoped>
    .detail-wrap {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;

        padding-top: 46px;

        overflow: auto;
        -webkit-overflow-scrolling: touch;
    }
</style>