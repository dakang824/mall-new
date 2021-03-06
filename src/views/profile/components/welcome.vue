<!--
 * @Author: yukang 1172248038@qq.com
 * @Description: 欢迎页面
 * @Date: 2020-10-19 23:03:17
 * @LastEditTime: 2020-11-22 10:22:38
-->
<template>
  <div class="welcome">
    <el-container>
      <el-main>
        <div class="row">
          <div class="el-card">
            <el-image
              v-if="getMyInfo.icon"
              class="user_pic"
              :src="getMyInfo.icon | imgBaseUrl"
            />
            <el-image
              v-else
              :src="require('@/assets/imgs/header-avatar.png')"
            />

            <div>{{ getMyInfo.name }}</div>
            <div class="safe">
              <span>账号安全</span>
              <ul>
                <li
                  v-for="(item, index) in level"
                  :key="index"
                  :class="{ active: index == getMyInfo.accLevel }"
                >
                  {{ item }}
                </li>
              </ul>
            </div>
            <ol class="footer">
              <li v-if="getMyInfo.mobile">
                <el-image
                  :src="require('@/assets/imgs/profile-mobile.png')"
                ></el-image>
                已绑定
              </li>
              <li v-if="getMyInfo.status !== 1">
                <el-image
                  :src="require('@/assets/imgs/profile-no.png')"
                ></el-image>
                未开启
              </li>
            </ol>
          </div>
          <div class="el-card">
            <span>我的余额</span>
            <p>
              <i>￥</i>
              {{ getMyInfo.balance || 0 }}
            </p>
            <el-image
              :src="require('@/assets/imgs/profile-recharge.png')"
              @click="handleRecharge"
            ></el-image>
          </div>
          <div style="flex: 1">
            <ul class="block" :class="{ setWidth: !getCartItems.length }">
              <li class="el-card">
                <i>待付款</i>
                <p>{{ store.unpay_count || 0 }}</p>
              </li>
              <li class="el-card">
                <i>待收货</i>
                <p>{{ store.unRecive_count || 0 }}</p>
              </li>
              <li class="el-card">
                <i>退款</i>
                <p>{{ store.payback_count || 0 }}</p>
              </li>
              <li class="el-card">
                <i>待发货</i>
                <p>{{ store.unPost_count || 0 }}</p>
              </li>
            </ul>
          </div>
        </div>
        <div class="box">
          <i class="box__more" @click="handleMore">
            查看更多
            <span class="el-icon-d-arrow-right"></span>
          </i>
          <el-tabs
            type="border-card"
            class="el-card"
            @tab-click="handleTabsChange"
          >
            <el-tab-pane
              v-for="(item, index) in getTabsData"
              :key="index"
              :label="item.name"
            >
              <span v-if="index === 0" slot="label">{{ item.name }}</span>
              <ul class="block">
                <li
                  v-for="(it, ind) in item.data"
                  :key="ind"
                  @click.stop="handleItem(it)"
                >
                  <el-image
                    :src="
                      'pics' in it ? it.pics[0].path : it.logoPath | imgBaseUrl
                    "
                    fit="contain"
                    class="goods-detail-info__imgs"
                  />
                  <p>{{ it.name }}</p>
                </li>
              </ul>

              <empty
                icon="favor"
                :text="'暂无' + item.name"
                margin-top="42"
                :show="!item.data.length"
              ></empty>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-main>
      <el-aside v-if="getCartItems.length" width="230px" class="el-card">
        <div class="title">我的购物车</div>
        <div
          v-for="(it, ind) in getCartItems.slice(0, maxLen)"
          :key="ind"
          class="goods-item"
          @click="handleClick(it)"
        >
          <el-image
            :src="it.product.pics[0].path | imgBaseUrl"
            fit="cover"
          ></el-image>
          <div class="right">
            <p>{{ it.product.name }}</p>
            <span>￥{{ it.product.specList | minPrice }}</span>
          </div>
        </div>
        <div v-if="getCartItems.length > 5" class="more" @click="handleChange">
          <div v-if="maxLen == 5">
            查看全部(
            <span>{{ getCartItems.length }}</span>
            )
          </div>
          <div v-else>
            <i
              class="el-icon-d-arrow-right"
              style="transform: rotate(-90deg)"
            ></i>
          </div>
        </div>
      </el-aside>
    </el-container>
  </div>
</template>

<script>
  import { mapState } from "vuex";
  import Empty from "@/components/empty.vue";
  export default {
    name: "Welcome",
    components: { Empty },
    filters: {
      minPrice: (value) => {
        if (value.length) {
          return JSON.parse(JSON.stringify(value))
            .sort((a, b) => {
              return a.sellPrice - b.sellPrice;
            })[0]
            .sellPrice.toFixed(2);
        }
      },
    },
    inject: ["setCurrent"],
    data() {
      return {
        level: ["低", "中", "高"],
        maxLen: 5,
        current: 0,
      };
    },
    computed: {
      ...mapState({
        store: (state) => state.profileWelcome.store,
      }),
      getMyInfo() {
        return this.store.myInfo || {};
      },
      getCartItems() {
        return this.store.cartItems || [];
      },
      getTabsData() {
        return [
          {
            name: "商品收藏",
            data: this.store.collectProds,
          },
          { name: "店铺收藏", data: this.store.collectStores },
        ];
      },
    },
    async created() {
      await this.$store.dispatch("profileWelcome/getMyInfo", {});
    },
    methods: {
      handleRecharge() {
        this.setCurrent("Recharge");
      },
      handleMore() {
        this.setCurrent(this.current === 0 ? "FavorGoods" : "FavorGoods");
      },
      handleTabsChange(e) {
        this.current = Number(e.index);
      },
      handleChange(e) {
        this.maxLen = this.maxLen === 5 ? this.getCartItems.length : 5;
      },
      handleItem(e) {
        if ("pics" in e) {
          this.handleClick(e);
        } else {
          this.$router.push({
            path: "/store",
            query: { id: e.id },
          });
        }
      },
      handleClick(e) {
        let type, id;
        if ("product" in e) {
          type = e.product.type;
          id = e.product.id;
        } else {
          type = e.type;
          id = e.id;
        }
        window.open(`#/goods-detail?type=${type}&id=${id}`);
        // this.$router.push({
        //   path: "/goods-detail",
        //   query: { type: e.type, id: e.id },
        // });
        // this.getData();
      },
    },
  };
</script>

<style lang="scss" scoped>
  @import "@/assets/scss/settings";

  .welcome {
    .el-main {
      > .row {
        > .el-card:first-child {
          min-width: 260px;
          padding: $padding;
          padding-bottom: 5px;
          text-align: center;

          .user_pic {
            width: 60px;
            height: 60px;
          }

          .safe {
            @include center-flex(y);

            justify-content: center;
            padding: $padding 0 0;
            margin: $padding 0 28px 0;
            border-top: 1px solid $colorBorder;

            ul {
              @include justify();

              li {
                width: 43px;
                height: 20px;
                margin-left: 10px;
                line-height: 20px;
                border: 1px solid #333;
                border-radius: 25px;

                &.active {
                  color: #fff;
                  background: $green;
                  border-color: $green;
                }
              }
            }
          }

          .footer {
            @include justify();

            li {
              @include center-flex(y);

              .el-image {
                margin: 6px;
              }
            }
          }
        }

        > .el-card:nth-child(2) {
          min-width: 190px;
          margin: 0 $padding;
          text-align: center;

          span {
            display: block;
            padding: 35px 0 30px;
          }

          p {
            margin-bottom: 25px;
            font-size: $text-large;
            font-weight: bold;
            color: $green;

            i {
              font-size: $text-x-small;
            }
          }

          .el-image {
            cursor: pointer;
          }
        }
      }

      .row {
        @include justify();

        .block {
          @include justify();

          &.setWidth {
            .el-card {
              min-width: 48.9%;
            }
          }

          flex-wrap: wrap;

          .el-card {
            min-width: 47.9%;
            height: 100px;
            padding-top: 25px;
            text-align: center;

            p {
              font-size: 20px;
              color: $green;
            }

            &:first-child,
            &:nth-child(3) {
              margin-right: $padding;
            }

            &:first-child,
            &:nth-child(2) {
              margin-bottom: $padding;
            }
          }
        }
      }

      .box {
        position: relative;

        &__more {
          position: absolute;
          right: 0;
          z-index: 99;
          height: 53px;
          padding-right: 10px;
          line-height: 53px;
          cursor: pointer;

          &:hover {
            color: $green;
          }
        }

        ::v-deep {
          .el-tabs {
            margin-top: $padding;

            &__content {
              padding: $padding;
            }

            .block {
              @include center-flex(y);

              flex-wrap: wrap;
              margin-left: 25px;

              li {
                width: 89px;
                margin-top: 49px;
                margin-right: 49px;
                text-align: center;
                cursor: pointer;

                &:nth-child(5n) {
                  margin-right: 0;
                }

                .el-image {
                  width: 89px;
                  height: 89px;
                  border: 1px solid $colorBorder;
                  border-radius: 50px;
                }

                p {
                  margin: 22px 0 46px;

                  @include ellipsis-lines(1);
                }
              }
            }

            .el-tabs__item {
              position: relative;
              width: 297px;
              height: 54px;
              padding: 0;
              font-size: $text-x-small;
              font-weight: bold;
              line-height: 54px;
              color: $color5;
              text-align: center;

              &:hover {
                color: $green;
              }

              &.is-active {
                color: $green;

                &::after {
                  position: absolute;
                  top: 0;
                  left: 0;
                  display: block;
                  width: 100%;
                  height: 4px;
                  content: "";
                  background: $green;
                }
              }
            }

            &--border-card {
              & > .el-tabs__header {
                background-color: #d4d4d4;
              }
            }
          }
        }

        .empty {
          margin-bottom: 60px;
        }
      }
    }

    .el-aside {
      padding: $padding 0;
      margin-left: $padding;

      .title {
        @include center-flex(y);

        margin: 0 0 $padding $padding;

        &::before {
          display: block;
          width: 2px;
          height: 14px;
          margin-right: 14px;
          content: "";
          background: $green;
        }
      }

      .right {
        display: flex;
        flex-direction: column;
        justify-content: space-between;

        p {
          @include ellipsis-lines(2);
        }
      }

      .goods-item {
        display: flex;
        padding: 5px $padding;
        cursor: pointer;

        &:hover {
          background: rgba(0, 0, 0, 0.1);
        }

        &:last-child {
          margin-bottom: 0;
        }

        .el-image {
          width: 80px;
          min-width: 80px;
          height: 80px;
          margin-right: $padding;
        }

        span {
          font-weight: bold;
          color: $error;
        }
      }

      .more {
        padding: 3px 0 0;
        text-align: center;
        cursor: pointer;

        span {
          color: $green;
        }
      }
    }
  }
</style>
