<script setup>
import { ref, onMounted } from 'vue'
import { gsap } from "gsap"
const photobox = ref({
  container: null,
  img_data: [],
  container_width: 0,
  container_height: 0,
  photo_width: 0,
  photo_height: 0,
  if_movable: false,
  mouse_x: 0,
  mouse_y: 0,
  standard_width: 1440,
  scale_nums: 1,
  
  init() {
    this.resize()
    window.addEventListener("resize", this.resize.bind(this))
    this.container.addEventListener("mousedown", this.handleMouseDown.bind(this))
    this.container.addEventListener("mouseup", this.handleMouseUp.bind(this))
    this.container.addEventListener("mouseleave", this.handleMouseLeave.bind(this))
    this.container.addEventListener("mousemove", this.handleMouseMove.bind(this))
  },
  
  resize() {
    const imgs = [...this.container.querySelectorAll(".photos_line_photo")]
    if (imgs.length === 0) return
    
    this.container_width = this.container.offsetWidth
    this.container_height = this.container.offsetHeight
    this.photo_width = imgs[0].offsetWidth
    this.photo_height = imgs[0].offsetHeight
    this.scale_nums = document.body.offsetWidth / this.standard_width
    this.container.style.transform = `scale(${this.scale_nums})`
    
    gsap.to(imgs, {
      transform: `translate(0,0)`,
      duration: 0,
      ease: 'power4.out'
    })
    
    this.img_data = imgs.map(img => ({
      node: img,
      x: img.offsetLeft,
      y: img.offsetTop,
      mov_x: 0,
      mov_y: 0,
      ani: null
    }))
  },
  
  handleMouseDown(event) {
    this.if_movable = true
    this.mouse_x = event.clientX
    this.mouse_y = event.clientY
  },
  
  handleMouseUp() {
    this.if_movable = false
  },
  
  handleMouseLeave() {
    this.if_movable = false
  },
  
  handleMouseMove(event) {
    if (!this.if_movable || !this.img_data) return
    
    const x = event.clientX
    const y = event.clientY
    const distance_x = (x - this.mouse_x) / this.scale_nums
    const distance_y = (y - this.mouse_y) / this.scale_nums
    
    this.img_data.forEach((img) => {
      let duration = 1
      img.mov_x += distance_x
      
      if (img.x + img.mov_x > this.container_width) {
        img.mov_x -= this.container_width
        duration = 0
      }
      if (img.x + img.mov_x < -this.photo_width) {
        img.mov_x += this.container_width
        duration = 0
      }
      
      img.mov_y += distance_y
      if (img.y + img.mov_y > this.container_height) {
        img.mov_y -= this.container_height
        duration = 0
      }
      if (img.y + img.mov_y < -this.photo_height) {
        img.mov_y += this.container_height
        duration = 0
      }
      
      if (img.ani) img.ani.kill()
      
      img.ani = gsap.to(img.node, {
        transform: `translate(${img.mov_x}px,${img.mov_y}px)`,
        duration: duration,
        ease: 'power4.out'
      })
    })
    
    this.mouse_x = x
    this.mouse_y = y
  }
})
onMounted(() => {
  photobox.value.container = document.querySelector(".photos")
  if (photobox.value.container) {
    photobox.value.init()
  }
})

const config = {
  totalImages: 28,      // 总图片数量
  imagesPerRow: 7       // 每行图片数量
}

// 计算需要多少行
const rows = Math.ceil(config.totalImages / config.imagesPerRow)

// 生成图片数据数组
const images = Array.from({ length: config.totalImages }, (_, i) => ({
  id: i + 1,
  src: `/src/assets/photos/photo (${i + 1}).png`
}))


</script>

<template>
  <div class="photos">
    <div 
      class="photos_line" 
      v-for="(row, rowIndex) in Array.from({ length: rows })" 
      :key="rowIndex"
    >
      <div 
        class="photos_line_photo" 
        v-for="image in images.slice(rowIndex * config.imagesPerRow, (rowIndex + 1) * config.imagesPerRow)" 
        :key="image.id"
      >
        <img :src="image.src" />
      </div>
    </div>
  </div>
</template>


<style scoped>
.photos {
  width: 100%;
  overflow: hidden;
}

.photos_line {
  display: flex;
  white-space: nowrap;
}

.photos_line_photo {
  flex-shrink: 0;
  margin: 0 10px;
}

.photos_line_photo img {
  max-width: 100%;
  height: auto;
  display: block;
}
</style>
