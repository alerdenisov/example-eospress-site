<template lang="pug">
  section.page   
    .page-overlay
      .page-overlay__hash {{ contentHash }} 
    .modal.fade.show(tabindex='-1', role='dialog', aria-labelledby='exampleModalLongTitle', aria-hidden='true' style="padding-left: 0; background-color: #0000009c;")
      .modal-dialog(role='document')
        .modal-content
          .modal-header
            h5.modal-title Attention!
          .modal-body 
            p Providen content was changed and validation is failed.
            pre
              code.code Recovered PKey is 
              br
              code.code.code--high EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
              br
              code.code isn't key of 
              code.code.code--high eosio
          .modal-footer
            //- button.btn.btn-secondary(type='button', data-dismiss='modal') Close
            button.btn.btn-danger(type='button') Continue browsing
    .page__header.page-header.page-header--parachute(style="background-image: url('/images/bg.jpg')")
      .container
        .row
          .col-lg-12
            h1.page-header__title {{ mainNode.title }}

    .page__content.page-content
      .page-content__box.page-box.page-box--about-me.container
        .row.align-items-center
          .col-lg-4(style="margin: -180px 0px")
            img.img-fluid(src='/images/photo-1.jpg')
          .col-lg-8
            h3 {{ mainNode.aboutTitle }}
            p {{ mainNode.aboutText }}
      .page-content__box.page-box.page-box--fundraising.box-fundraising.container(style="position: relative; z-index: 1")
        .row.align-items-center
          .col-lg-12(style="margin-bottom: 30px;")
            h2.box-fundraising__title Donate for a jump
            p.box-fundraising__text 
              span Send your token to 
              span.box-fundraising__wallet(@click='hackIt' v-if='!hacking') {{ mainNode.wallet }}
              input.box-fundraising__hack-wallet(v-model='mainNode.wallet' v-else)
              span  to make jump true

      .page-content__box.page-box.page-box--story.box-story.container
        .row.justify-content-center
          .col-lg-8
            h3(style="margin-bottom: 30px") Story
            p(v-for="p in stories") {{ p }}

      .page-content__box.page-box.page-box--share.box-share.container
        .row.justify-content-center
          .box-share__row.col(style="text-align: center; display: flex; justify-content: center; align-items: center")
            h3 Share this story
            button.page-button.page-button--facebook Facebook
            button.page-button.page-button--steemit Steemit
            button.page-button.page-button--email Email to friend

      
      .page-content__box.page-box.page-box--activity.container
        .row
          .col-lg-4
            .page-box--supporters.box-supporters
              .row
                .col
                  h2.box-supporters__title Supporters
                  .supporter(v-for="item in supporters")
                    .supporter__column
                      img.supporter__avatar(:src="item.photo")
                    .supporter__column.supporter__grow
                      .supporter__row.supporter__grow
                        span.supporter__name.supporter__grow {{ item.name }}
                        span.supporter__meta {{ agoIt(item.date) }}
                      p.supporter__message {{ item.message }}
                      span.supporter__amount {{ item.amount }}
          .col-lg-8
            .page-box--posts.box-posts.container
              .row
                .col
                  h2.box-posts__title News Feed
                  .post(v-for="item in posts")
                    div.row
                      .col-lg-4
                        img.img-fluid(:src="item.photo")
                      .col-lg-8
                        span.post__meta {{ agoIt(item.date) }}
                        p(v-for="line in item.lines") {{ line }}


</template>

<script>
const Eos = require("eosjs");
const moment = require("moment");
const { ecc } = Eos.modules;

export default {
  data: () => ({
    hacking: false
  }),

  computed: {
    parsedContent() {
      return JSON.stringify({
        mainNode: this.mainNode,
        storyFeed: this.storyFeed,
        posts: this.posts,
        supporters: this.supporters
      });
    },
    contentHash() {
      return Eos.modules.ecc.sha256(this.parsedContent);
    },
    eos() {
      return Eos();
    },
    supporters() {
      if (!this.supportersFeed) {
        return [];
      }

      return this.supportersFeed.rows.map(feedRecord =>
        JSON.parse(feedRecord.content)
      );
    },
    posts() {
      if (!this.postsFeed) {
        return [];
      }

      return this.postsFeed.rows.map(postRecord =>
        JSON.parse(postRecord.content)
      );
    },
    mainNode() {
      if (!this.mainFeed) {
        return {};
      }

      return JSON.parse(this.mainFeed.rows[0].content);
    },
    stories() {
      if (!this.storiesFeed) {
        return [];
      }

      return this.storiesFeed.rows.map(story => story.content);
    }
  },

  asyncComputed: {
    async supportersFeed() {
      return await this.eos.getTableRows(
        true,
        "oldpa",
        "suppfeed",
        "contentnode"
      );
    },
    async postsFeed() {
      return await this.eos.getTableRows(
        true,
        "oldpa",
        "postsfeed",
        "contentnode"
      );
    },
    async storiesFeed() {
      return await this.eos.getTableRows(
        true,
        "oldpa",
        "storiesfeed",
        "contentnode"
      );
    },
    async mainFeed() {
      return await this.eos.getTableRows(
        true,
        "oldpa",
        "mainfeed",
        "contentnode"
      );
    }
  },

  methods: {
    agoIt(date) {
      return moment(date).fromNow();
    },
    hackIt() {
      this.hacking = !this.hacking;
    }
  }
};
</script>

<style>

</style>
