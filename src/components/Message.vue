<template>
  <div>
    <!-- シェアを全て表示する -->
    <div v-for="(value, index) in shares" :key="index">
      <div class="message">
        <div class="flex">
          <!-- シェア投稿ユーザの名前表示 -->
          <p class="name">{{ value.name }}</p>
          <!-- いいねボタン -->
          <img class="icon" src="../assets/heart.png" @click="fav(index)" alt />
          <!-- いいね数表示 -->
          <p class="number">{{ value.like.length }}</p>
          <!-- シェア削除 -->
          <img
            class="icon"
            src="../assets/cross.png"
            @click="del(index)"
            alt
            v-if="path && profile"
          />
          <!-- シェア詳細画面 -->
          <img
            class="icon detail"
            src="../assets/detail.png"
            @click="
              $router.push({
                path: '/detail/' + value.item.id,
                params: { id: value.item.id },
              })
            "
            alt
            v-if="profile"
          />
          <!-- コメント数表示 -->
          <p class="number" v-if="profile">{{ value.comment.length }}</p>
        </div>
        <!-- シェア投稿内容表示 -->
        <p class="text">{{ value.item.share }}</p>
        <!-- シェア投稿時間表示 -->
        <p class="text time">{{ value.item.updated_at | moment }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
  // Messageコンポーネントに渡されたIDの値を受ける
  props: ["id"],
  data() {
    return {
      shares: [],
      path: true,
      profile: true,
    };
  },
  // 日付フィルタを設定
  filters: {
    moment: function (date) {
      return moment(date).format("YYYY/MM/DD HH:mm:ss");
    },
  },
  methods: {
    fav(index) {
      // 対象のシェアにいいねがついているか判定
      const result = this.shares[index].like.some((value) => {
        return value.user_id == this.$store.state.user.id;
      });
      // 自分がすでにいいねしている場合
      if (result) {
        // 自分がすでにいいねしている場合、自分のいいねを削除する
        this.shares[index].like.forEach((element) => {
          if (element.user_id == this.$store.state.user.id) {
            axios({
              method: "delete",
              url: "https://aqueous-tor-62904.herokuapp.com/api/like",
              data: {
                share_id: this.shares[index].item.id,
                user_id: this.$store.state.user.id,
              },
            }).then((response) => {
              console.log(response);
              this.$router.go({
                path: this.$router.currentRoute.path,
                force: true,
              });
            });
          }
        });
        // 自分がすでにいいねしていない場合
      } else {
        // いいねを追加
        axios
          .post("https://aqueous-tor-62904.herokuapp.com/api/like", {
            share_id: this.shares[index].item.id,
            user_id: this.$store.state.user.id,
          })
          .then((response) => {
            console.log(response);
            this.$router.go({
              path: this.$router.currentRoute.path,
              force: true,
            });
          });
      }
    },
    // シェア削除
    del(index) {
      // 自分のシェアだけ消せるようにする
      if (this.shares[index].item.user_id == this.$store.state.user.id) {
        axios
          .delete(
            "https://aqueous-tor-62904.herokuapp.com/api/shares/" +
              this.shares[index].item.id
          )
          .then((response) => {
            console.log(response);
            console.log(
              this.shares[index].item.user_id + " " + this.$store.state.user.id
            );
            this.$router.go({
              path: this.$router.currentRoute.path,
              force: true,
            });
          });
      } else {
        alert("自分のシェアではありません");
        console.log(
          this.shares[index].item.user_id + " " + this.$store.state.user.id
        );
      }
    },
    // 画面表示時のシェア表示
    async getShares() {
      let data = [];
      const shares = await axios.get(
        "https://aqueous-tor-62904.herokuapp.com/api/shares"
      );
      //シェア表示
      await Promise.all(
        shares.data.data.map((d) => {
          axios
            // シェアidごとシェア情報を取得する
            .get("https://aqueous-tor-62904.herokuapp.com/api/shares/" + d.id)
            .then((response) => {
              // プロフィール画面の場合
              if (this.$route.name == "profile") {
                // シェアのIDとログイン中のユーザIDが一致する場合、シェア情報表示
                if (response.data.item.user_id == this.$store.state.user.id) {
                  data.push(response.data);
                }
                // 詳細画面の場合
              } else if (this.$route.name == "detail") {
                // シェアのIDと詳細画面のIDが一致する場合シェア情報表示
                if (response.data.item.id == this.id) {
                  data.push(response.data);
                }
                // プロフィール・詳細画面以外の場合全件表示
              } else {
                data.push(response.data);
              }
            });
        })
      );
      this.shares = data;
      console.log(this.shares);
    },
  },
  created() {
    if (this.$route.name === "home") {
      this.path = false;
    }
    if (this.$route.name === "detail") {
      this.profile = false;
    }
    this.getShares();
  },
};
</script>

<style scoped>
.flex {
  display: flex;
}
.icon {
  width: 25px;
  height: 25px;
}
.detail {
  margin-left: 50px;
}
.message {
  padding: 20px;
  border-bottom: solid 1px white;
  border-left: solid 1px white;
  /* 文字数が長い場合に折り返し */
  word-wrap: break-word;
}
.name {
  font-size: 18px;
  font-weight: bold;
  margin-right: 10px;
}
.text {
  margin-top: 10px;
}
.number {
  margin-left: 10px;
  margin-right: 10px;
}
.time {
  font-size: 12px;
}
</style>