<!--
 * @Author: yukang 1172248038@qq.com
 * @Description: 商品收藏
 * @Date: 2020-10-29 16:20:52
 * @LastEditTime: 2020-10-31 15:41:36
-->
<template>
  <div class="favor-goods el-card">
    <el-tabs value="first">
      <el-tab-pane label="店铺收藏" name="first">
        <div v-loading="listLoading" class="favor-goods__box">
          <el-card v-for="(item, index) in list" :key="index" class="box-card">
            <div @click="handleGoDetail(item)">
              <el-image
                :src="item.logoPath | imgBaseUrl"
                fit="scale-down"
              ></el-image>
              <div class="box-card__content">
                <p>{{ item.name }}</p>
              </div>
            </div>
          </el-card>
        </div>
        <el-pagination
          v-if="list.length"
          background
          :current-page="pageNum"
          :page-size="pageSize"
          :layout="layout"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        ></el-pagination>

        <empty
          icon="favor"
          text="暂无店铺收藏"
          margin-top="90"
          :show="!list.length && !listLoading"
        ></empty>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
  import { getMyCollectStore } from "@/api/profile";
  import Empty from "@/components/empty.vue";
  export default {
    name: "FavorStore",
    components: { Empty },
    data() {
      return {
        list: [],
        layout: "total, sizes, prev, pager, next, jumper",
        total: 0,
        listLoading: true,
        pageNum: 1,
        pageSize: 10,
      };
    },
    created() {
      this.fetchData();
    },
    methods: {
      handleSizeChange(val) {
        this.pageSize = val;
        this.fetchData();
      },
      handleCurrentChange(val) {
        this.pageNum = val;
        this.fetchData();
      },
      async fetchData() {
        this.listLoading = true;
        const {
          data: { collectStores },
        } = await getMyCollectStore({
          pageNum: this.pageNum,
          pageSize: this.pageSize,
        });
        this.list = collectStores.list;
        this.total = collectStores.total;
        setTimeout(() => {
          this.listLoading = false;
        }, 300);
      },
      handleGoDetail(e) {
        const { id, type } = e;
        this.$router.push({
          path: "/store",
          query: { id },
        });
      },
    },
  };
</script>

<style lang="scss" scoped>
  @import "@/assets/scss/settings";

  .favor-goods {
    &__box {
      @include center-flex(y);

      flex-wrap: wrap;
    }

    min-height: 529px;

    .box-card {
      width: 172px;

      .el-image {
        background: $imgbg;
      }

      &__content {
        padding: $padding;
        text-align: center;
      }
    }
  }

  .el-image {
    width: 172px;
    height: 172px;
  }

  ::v-deep {
    .el-pagination {
      padding: 20px;
      text-align: center;
    }

    .el-card {
      margin: 0 18px 18px 0;
      cursor: pointer;

      &__body {
        padding: 0;
      }

      &:nth-child(5n) {
        margin-right: 0;
      }
    }

    .el-tabs {
      &__item {
        width: 150px !important;
        height: 46px;
        font-size: $text-x-small;
        line-height: 46px;
        text-align: center;
      }

      .el-tab-pane {
        padding: 0 $padding;
      }
    }
  }
</style>
