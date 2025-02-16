<template>
  <div :class="{ dark: isDark }" class="wrapper">
    <div class="buttons">
      <button @click="getRandomArticle" class="new-article">New Article</button>
      <a :href="url" target="_blank" class="read-more">Read more</a>
    </div>
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
    }, 200);
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
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f7f1e9;
  min-height: 100vh;
  padding: 3px;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
}

.container {
  width: 100%;
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
}

.summary {
  font-size: 18px;
  word-wrap: break-word;
  text-align: start;
  text-indent: 2rem;
}

.buttons {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 20px;
}

.new-article,
.read-more {
  padding: 10px 20px;
  background-color: #71a0d3;
  color: #fff;
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
    padding-top: 60px;
  }
  
  .title {
    font-size: 22px;
  }
  
  .description {
    font-size: 12px;
  }
  
  .summary {
    font-size: 16px;
  }
  
  .theme-toggle img {
    width: 35px;
  }
  
  .new-article,
  .read-more {
    padding: 5px 10px;
    font-size: 11px;
  }
}
</style>

