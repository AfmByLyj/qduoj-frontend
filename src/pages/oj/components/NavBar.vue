<template>
  <div id="header">
    <Menu theme="light" mode="horizontal" @on-select="handleRoute" :active-name="activeMenu" class="oj-menu">
      <div class="logo"><span>{{website.website_name}}</span></div>
      <div class="items">
        <Menu-item name="/">
          <Icon type="home"></Icon>
          {{$t('m.Home')}}
        </Menu-item>
        <Menu-item name="/problem">
          <Icon type="ios-keypad"></Icon>
          {{$t('m.NavProblems')}}
        </Menu-item>
        <Menu-item name="/contest">
          <Icon type="trophy"></Icon>
          {{$t('m.Contests')}}
        </Menu-item>
        <Menu-item name="/status">
          <Icon type="ios-pulse-strong"></Icon>
          {{$t('m.NavStatus')}}
        </Menu-item>
        <Menu-item name="/anime">
          <Icon type="social-vimeo"></Icon>
          Anime
        </Menu-item>
        <Submenu name="rank">
          <template slot="title">
            <Icon type="podium"></Icon>
            {{$t('m.Rank')}}
          </template>
          <Menu-item name="/rl-rank">
            {{$t('m.RL_Rank')}}
          </Menu-item>
          <Menu-item name="/acm-rank">
            {{$t('m.ACM_Rank')}}
          </Menu-item>
          <!-- <Menu-item name="/oi-rank">
            {{$t('m.OI_Rank')}}
          </Menu-item> -->
        </Submenu>
        <Submenu name="about">
          <template slot="title">
            <Icon type="information-circled"></Icon>
            {{$t('m.About')}}
          </template>
          <Menu-item name="/about">
            {{$t('m.Judger')}}
          </Menu-item>
          <Menu-item name="/FAQ">
            {{$t('m.FAQ')}}
          </Menu-item>
        </Submenu>
      </div>
      <template v-if="!isAuthenticated">
        <div class="btn-menu">
          <Button type="ghost"
                  ref="loginBtn"
                  shape="circle"
                  @click="handleBtnClick('login')">{{$t('m.Login')}}
          </Button>
          <Button v-if="website.allow_register"
                  type="ghost"
                  shape="circle"
                  @click="handleBtnClick('register')"
                  style="margin-left: 5px;">{{$t('m.Register')}}
          </Button>
        </div>
      </template>
      <template v-else>
        <Dropdown class="drop-menu" @on-click="handleRoute" placement="bottom" trigger="click">
          <Button type="text" class="drop-menu-title">
            <div class="fixed">
              <img class="avatar" :src="profile.avatar">
              <div>
                <span v-if="profile.userSpan !== ''" v-html="profile.userSpan.replace('|', profile.user.username)"></span>
                <span v-else>{{ profile.user.username }}</span>
                <Icon type="arrow-down-b"></Icon>
              </div>
            </div>
          </Button>
          <Dropdown-menu slot="list">
            <Dropdown-item name="/user-home">{{$t('m.MyHome')}}</Dropdown-item>
            <Dropdown-item name="/status?myself=1">{{$t('m.MySubmissions')}}</Dropdown-item>
            <Dropdown-item name="/setting/profile">{{$t('m.Settings')}}</Dropdown-item>
            <Dropdown-item v-if="isAdminRole" name="/admin">{{$t('m.Management')}}</Dropdown-item>
            <Dropdown-item divided name="/logout">{{$t('m.Logout')}}</Dropdown-item>
          </Dropdown-menu>
        </Dropdown>
      </template>
    </Menu>
    <Modal v-model="modalVisible" :width="400">
      <div slot="header" class="modal-title">{{$t('m.Welcome_to')}} {{website.website_name_shortcut}}</div>
      <component :is="modalStatus.mode" v-if="modalVisible"></component>
      <div slot="footer" style="display: none"></div>
    </Modal>
  </div>
</template>

<script>
  import { mapGetters, mapActions } from 'vuex'
  import login from '@oj/views/user/Login'
  import register from '@oj/views/user/Register'

  export default {
    components: {
      login,
      register
    },
    mounted () {
      this.getProfile()
    },
    methods: {
      ...mapActions(['getProfile', 'changeModalStatus']),
      handleRoute (route) {
        if (route && route.indexOf('admin') < 0) {
          this.$router.push(route)
        } else {
          window.open('/admin/')
        }
      },
      handleBtnClick (mode) {
        this.changeModalStatus({
          visible: true,
          mode: mode
        })
      }
    },
    computed: {
      ...mapGetters(['website', 'modalStatus', 'user', 'isAuthenticated', 'isAdminRole', 'profile']),
      // 跟随路由变化
      activeMenu () {
        return '/' + this.$route.path.split('/')[1]
      },
      modalVisible: {
        get () {
          return this.modalStatus.visible
        },
        set (value) {
          this.changeModalStatus({visible: value})
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  #header {
    min-width: 300px;
    position: fixed;
    top: 0;
    left: 0;
    height: auto;
    width: 100%;
    z-index: 1000;
    background-color: #fff;
    box-shadow: 0 0 5px 0 rgba(0, 0, 0, 0.5);
    .oj-menu {
      background: #fdfdfd;
      height: auto;
      width: 80%;
      left: 10%;
    }

    .logo {
      text-align: center;
      font-size: 20px;
      float: left;
      width: 15%;
      line-height: 60px;
    }

    .items {
      position: relative;
    }

    .drop-menu {
      float: right;
      &-title {
        font-size: 18px;
        .fixed {
          display: flex;
          justify-content: center;
          align-items: center;
        }
        .avatar {
          display: block;
          height: 50px;
          width: 50px;
          border-radius: 50%;
          margin-right: 5px;
          box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
        }
      }
    }
    .btn-menu {
      font-size: 16px;
      float: right;
      margin-right: 10px;
    }
  }

  .modal {
    &-title {
      font-size: 18px;
      font-weight: 600;
    }
  }
</style>
