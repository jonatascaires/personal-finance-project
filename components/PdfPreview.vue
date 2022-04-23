<template>
  <div class="pdf-preview-container">
    <div
      v-for="page in docPages"
      :key="page"
      ref="container"
      class="page-container flex justify-center"
      :style="{
        width: `600px`,
      }"
    >
      <canvas v-if="renderList.includes(page)"> </canvas>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    url: {
      type: String,
      required: true,
    },
    renderPages: {
      type: Number,
      default: 5,
    },
    customScroll: {
      type: Boolean,
      default: false,
    },
    offsetHeight: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      doc: null,
      docPages: 0,
      currentPage: 0,
      pageHeight: 0,
      renderList: [],
    }
  },
  watch: {
    url: {
      immediate: true,
      handler() {
        this.getPDFFile()
      },
    },
  },
  mounted() {
    if (!this.customScroll) {
      document.addEventListener('scroll', this.scroll)
    }
  },
  beforeDestroy() {
    document.removeEventListener('scroll', this.scroll)
  },
  methods: {
    getPDFFile() {
      if (!this.url) return
      this.currentPage = 0
      const pdfJS = require('pdfjs-dist/build/pdf')
      const pdfjsWorker = require('pdfjs-dist/build/pdf.worker.entry')
      pdfJS.GlobalWorkerOptions.workerSrc = pdfjsWorker
      pdfJS.getDocument(this.url).then(pdf => {
        // console.log('pdf: ', pdf)
        this.doc = pdf
        this.docPages = pdf._pdfInfo.numPages
        this.$nextTick(() => {
          this.docPages && this.scrollToPage(1)
        })
      })
    },
    scrollToPage(pageNo) {
      if (this.currentPage === pageNo) return
      this.currentPage = pageNo
      let list = []
      for (
        let page = pageNo - this.renderPages;
        page <= pageNo + this.renderPages;
        page++
      ) {
        list.push(page)
      }
      list = list.filter(page => page <= this.docPages && page >= 1)
      this.$nextTick(() => {
        this.renderList = list
        this.renderList.forEach(page => {
          this.renderPage(page)
        })
      })
    },
    // Render page
    renderPage(pageNo) {
      this.doc.getPage(pageNo).then(page => {
        // console.log('page: ', page)
        const container = this.$refs.container[pageNo - 1]
        if (!container) return
        const canvas = container.querySelector('canvas')
        if (!canvas || canvas.__rendered) return
        const ctx = canvas.getContext('2d')
        const dpr = window.devicePixelRatio || 1
        const bsr =
          ctx.webkitBackingStorePixelRatio ||
          ctx.mozBackingStorePixelRatio ||
          ctx.msBackingStorePixelRatio ||
          ctx.oBackingStorePixelRatio ||
          ctx.backingStorePixelRatio ||
          1
        const ratio = dpr / bsr
        const rect = container.getBoundingClientRect()
        const viewport = page.getViewport(1)
        const width = rect.width
        const height = (width / viewport.width) * viewport.height
        canvas.style.width = `${width}px`
        canvas.style.height = `${height}px`
        this.pageHeight = height
        canvas.height = height * ratio
        canvas.width = width * ratio
        ctx.setTransform(ratio, 0, 0, ratio, 0, 0)
        page.render({
          canvasContext: ctx,
          viewport: page.getViewport(width / viewport.width),
        })
        canvas.__rendered = true
      })
    },
    scroll() {
      this.checkRender(document.documentElement)
    },
    checkRender(el) {
      if (!this.pageHeight) return
      let scrollTop = el.scrollTop
      if (el === document.documentElement) {
        scrollTop =
          el.scrollTop || window.pageYOffset || document.body.scrollTop
      }
      let page = Math.floor((scrollTop - this.offsetHeight) / this.pageHeight)
      page = Math.max(page, 1)
      page = Math.min(page, this.docPages)
      this.scrollToPage(page)
    },
  },
}
</script>