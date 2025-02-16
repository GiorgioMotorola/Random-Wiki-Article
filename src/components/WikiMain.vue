<template>
  <div :class="{ dark: isDark }" class="wrapper">
    <div class="app-title">RAND.WIKI&#9860;</div>
    <div class="buttons">
      <button @click="getRandomArticle" class="new-article">Random</button>
      <button :href="url" target="_blank" class="read-more">Read More</button>
    </div>
    <div class="desc">click "Random" or swipe right if you're on mobile</div>
    <button @click="toggleTheme" class="theme-toggle">
      <img 
        src="/public/images/night2.png" 
        alt="Theme toggle icon" 
        class="toggle-icon"
        :class="{ rotated: isDark }" />
    </button>
    <transition name="slide">
      <div
        :key="articleKey"
        class="container"
        @touchstart="handleTouchStart"
        @touchmove="handleTouchMove"
        @touchend="handleTouchEnd"
        @click="openArticle"
        :style="{ transform: `translateX(${translateX}px)`, transition: touchTransition }"
      >
        <div v-if="loading" class="skeleton-overlay">
          <div class="skeleton-line title-skeleton"></div>
          <div class="skeleton-line description-skeleton"></div>
          <div class="skeleton-line image-skeleton"></div>
          <div class="skeleton-line summary-skeleton"></div>
        </div>
        <div class="body-container">
          <div class="title">{{ title }}</div>
          <div class="description">{{ description }}</div>
          <img class="wiki-image" v-if="image" :src="image" :alt="title" />
          <div class="summary">{{ summary }}</div>
        </div>
      </div>
    </transition>
  </div>
</template>


<script>
export default {
  data() {
    return {
      title: "",
      description: "",
      summary: "",
      image: "",
      url: "",
      isDark: false,
      touchStartX: 0,
      translateX: 0,
      touchTransition: "none",
      articleKey: 0, 
      swipeThreshold: 120,
      loading: false,
    };
  },
  methods: {
    getRandomArticle() {
  this.loading = true;

  fetch("https://en.wikipedia.org/api/rest_v1/page/random/summary")
    .then((res) => res.json())
    .then((data) => {
      this.title = data.title;
      this.description = data.description || "No description available";
      this.summary = data.extract;
      this.image = data.thumbnail ? data.thumbnail.source : "";
      this.url = data.content_urls.desktop.page;
      this.articleKey++;
      this.translateX = window.innerWidth;

      setTimeout(() => {
  this.translateX = 0; 
  this.loading = false;
}, 100);

    })
    .catch((err) => {
      console.error(err);
      this.loading = false;
    });
},

    toggleTheme() {
      this.isDark = !this.isDark;
    },
    openArticle() {
      if (this.url) {
        window.open(this.url, "_blank");
      }
    },
    handleTouchStart(e) {
      this.touchStartX = e.changedTouches[0].clientX;
      this.touchTransition = "none";
    },
    handleTouchMove(e) {
      const currentX = e.changedTouches[0].clientX;
      this.translateX = currentX - this.touchStartX;
    },
    handleTouchEnd() {
  this.touchTransition = "transform 0.3s ease-out";
  
  if (this.translateX < -this.swipeThreshold) {
    this.loading = true;
    this.translateX = -window.innerWidth;
    
    setTimeout(() => {
      this.getRandomArticle();
    }, 100);
  } else {
    this.translateX = 0;
  }
},
  },
  mounted() {
    this.getRandomArticle();
  },
};
</script>


<style scoped>

.app-title {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 30px;
  color: #649cd8;
  font-weight: 550;
}

.desc {
  font-size: 15px;
  color: rgb(122, 126, 126);
  font-style: italic;
  margin-bottom: 1.5rem;
  font-family: Arial, Helvetica, sans-serif;
}


.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #fdfdfd;
  min-height: 100vh;
  padding: 3px;
  font-family: Arial, Helvetica, sans-serif;
  overflow: hidden;
}

.container {
  width: 100vw;
  max-width: 500px;
  min-height: 300px; 
  background-color: #fff;
  border: 1px solid black;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  border-radius: 0.5rem;
  position: relative;
  padding-top: 60px;
  cursor: pointer;
  overflow: hidden;
  transition: transform 0.3s ease-out, opacity 0.2s ease-out;
  will-change: transform;
}

.container.loading {
  opacity: 0; 
}

.body-container {
  padding: 20px;
  text-align: center;
}

.skeleton-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 2;
  padding: 20px;
}

.skeleton-line {
  background: #e0e0e0;
  border-radius: 4px;
  margin-bottom: 10px;
  animation: pulse 3s infinite;
}

.title-skeleton {
  width: 60%;
  height: 24px;
}
.description-skeleton {
  width: 80%;
  height: 14px;
}
.image-skeleton {
  width: 100%;
  height: 200px;
}
.summary-skeleton {
  width: 100%;
  height: 18px;
}

.wiki-image {
  width: 95%;
  height: auto;
}

@keyframes pulse {
  0% { opacity: 1; }
  50% { opacity: 0.4; }
  100% { opacity: 1; }
}

.theme-toggle {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  cursor: pointer;
}

.theme-toggle img {
  width: 40px;
  transition: transform 0.3s ease;
}

.rotated {
  transform: rotate(180deg);
}

.dark {
  background-color: #0f0f0f;
  color: #fff;
}

.dark .container {
  background-color: #121212;
  border-color: #fff;
}

.title {
  font-size: 24px;
  word-wrap: break-word;
  margin-bottom: 10px;
}

.description {
  font-size: 14px;
  word-wrap: break-word;
  margin-bottom: 20px;
  font-style: italic;
}

.summary {
  font-size: 18px;
  word-wrap: break-word;
  text-align: start;
  text-indent: 2rem;
  margin-top: 2rem;
}

.buttons {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 20px;
}

.buttons button {
  padding: 3px 10px;
  background-color: #c5dfe6;
  border: none;
  border-radius: 5px;
  text-decoration: none;
  cursor: pointer;
  font-size: 16px;
  margin-bottom: 2rem;
}

.new-article:hover,
.read-more:hover {
  background-color: #0056b3;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.1s ease-out;
}
.slide-enter {
  transform: translateX(100%);
}
.slide-leave-to {
  transform: translateX(-100%);
}

@media only screen and (max-width: 600px) {
  .container {
    max-width: 100%;
    padding-top: 0px;
  }
  
  .title {
    font-size: 25px;
  }
  
  .description {
    font-size: 14px;
  }
  
  .summary {
    font-size: 14px;
  }
  
  .theme-toggle img {
    width: 35px;
  }
  
  .new-article,
  .read-more {
    font-size: 11px;
  }

  .desc {
  font-size: 13px;
}
}
</style>

