<template>
  <div class="Hackathon">
    <div class="logo-box" @click="$router.push({name:'home'})">
      <img src="../assets/imgs/RainbowDAOLogo.png" alt="">
    </div>
    <div class="title">
      RainbowDAO
    </div>
    <div class="sub-title">
      DAO Global Hackathon 2021
    </div>
    <div class="list">
      <a href="http://nftbank.rainbowdao.io" class="item" target="_blank">
        <div class="icon">
          <img src="../assets/imgs/hack_icon1.png" alt="">
        </div>
        <div class="name">
          DAO NFT Bank
        </div>
      </a>
      <a href="http://committee.rainbowdao.io" class="item" target="_blank">
        <div class="icon">
          <img src="../assets/imgs/hack_icon2.png" alt="">
        </div>
        <div class="name">
          DAO Multi-signature<br/>
          Management Committee
        </div>
      </a>
      <a href="http://multisign.rainbowdao.io" class="item" target="_blank">
        <div class="icon">
          <img src="../assets/imgs/hack_icon3.png" alt="">
        </div>
        <div class="name">
          Multi-signature Wallet
        </div>
      </a>
      <a href="http://factory.rainbowdao.io" class="item" target="_blank">
        <div class="icon">
          <img src="../assets/imgs/hack_icon4.png" alt="">
        </div>
        <div class="name">
          DAO Factory
        </div>
      </a>
    </div>
    <div class="footer">
      <div class="link">
        <a href="https://t.me/RainbowDAO" target="_blank">
          <img class="link-img" src="../assets/imgs/telegram.png" alt="">
        </a>
        <a href="https://twitter.com/RainbowDAOio" target="_blank">
          <img class="link-img" src="../assets/imgs/twitter.png" alt="">
        </a>
        <a href="https://medium.com/rainbowcity" target="_blank">
          <img class="link-img" src="../assets/imgs/medium.png" alt="">
        </a>
        <a href="https://discord.gg/vbnvFEeYRr" target="_blank">
          <img class="link-img" src="../assets/imgs/Discord.png" alt="">
        </a>
        <a href="https://github.com/RainbowDAO" target="_blank">
          <img class="link-img" src="../assets/imgs/github.png" alt="">
        </a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Hackathon"
}
</script>

<style lang="scss" scoped>
.Hackathon {
  width: 100vw;
  min-height: 600px;
  min-width: 1100px;
  height: 100vh;
  background: url("../static/imgs/hk_bg.png");
  background-size: 100% 100%;

  .logo-box {
    cursor: pointer;
    padding: 20px;
    text-align: center;

    img {
      width: 210px;
    }
  }

  .title {
    text-align: center;
    font-size: 90px;
    font-weight: bold;
    color: #ffffff;
    line-height: 150px;
  }

  .sub-title {
    font-size: 36px;
    font-weight: bold;
    text-align: center;
    color: #ff1d84;
    line-height: 60px;
  }

  .list {
    display: flex;
    flex-wrap: wrap;
    width: 1000px;
    margin: 20px auto;
    justify-content: space-between;

    .item {
      margin-top: 20px;
      width: 460px;
      padding: 20px;
      height: 150px;
      background: rgba(255, 255, 255, 0.20);
      border: 1px solid;
      border-image: linear-gradient(90deg, #12c2e9, #c471ed 55%, #f64f59) 1 1;
      display: flex;
      align-items: center;
      cursor: pointer;

      &:hover {
        box-shadow: 0px 0px 20px 3px rgba(218, 0, 127, 0.70);

        .name {
          color: #FF1D84;
        }
      }

      .icon {
        img {
          width: 60px;
          height: 60px;
        }
      }

      .name {
        margin-left: 10px;
        font-size: 28px;
        font-weight: 400;
        text-align: left;
        color: #ffffff;
        line-height: 36px;
      }
    }
  }

  .footer {
    text-align: center;

    .link {
      margin-top: 40px;

      .link-img {
        width: 24px;
        margin-right: 10px;
      }
    }
  }
}
</style>
