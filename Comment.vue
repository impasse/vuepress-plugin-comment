<template>
  <div></div>
</template>

<script>
import { provider, renderConfig, loadScript } from "./util";

const commentDomID = "vuepress-plugin-comment";
let timer;

export default {
  mounted() {
    const frontmatter = {
      to: {},
      from: {},
      ...this.$frontmatter,
    };
    clear() && needComment(frontmatter) && checkAndRenderComments(frontmatter);

    this.$router.afterEach((to, from) => {
      if (to && from && to.path === from.path) {
        return;
      }
      const frontmatter = {
        to,
        from,
        ...this.$frontmatter,
      };
      clearInterval(timer);
      clear() &&
        needComment(frontmatter) &&
        checkAndRenderComments(frontmatter).then(console.log);
    });
  },
};

/**
 * Clear last page comment dom
 */
function clear(frontmatter) {
  switch (COMMENT_CHOOSEN) {
    case "gitalk":
      return provider.gitalk.clear(commentDomID);
    case "valine":
      let el = COMMENT_OPTIONS.el || commentDomID;
      if (el.startsWith("#")) {
        el = el.slice(1);
      }
      return provider.valine.clear(el);
    default:
      return false;
  }
}

/**
 * Check if current page needs render comment
 */
function needComment(frontmatter) {
  return frontmatter.comment !== false && frontmatter.comments !== false;
}

function checkElement(selector, interval = 200, times = 10) {
  return new Promise((resolve) => {
    let el;
    setInterval(() => {
      if (--times > 0) {
        el = document.querySelector(selector);
        if (el) {
          clearInterval(timer);
          resolve(el);
        }
      } else {
        clearInterval(timer);
        resolve();
      }
    }, interval);
  });
}

function checkAndRenderComments(frontmatter) {
  return checkElement(COMMENT_CONTAINER).then((el) => {
    if (el) {
      switch (COMMENT_CHOOSEN) {
        case "gitalk":
          return provider.gitalk.render(frontmatter, commentDomID);
        case "valine":
          let el = COMMENT_OPTIONS.el || commentDomID;
          if (el.startsWith("#")) {
            el = el.slice(1);
          }
          return provider.valine.render(frontmatter, el);
        default:
          return false;
      }
    }
  });
}
</script>
