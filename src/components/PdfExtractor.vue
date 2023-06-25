<script lang="ts">
  import { Options, Vue } from 'vue-class-component'
  import * as pdfjsLib from 'pdfjs-dist'
import { TextItem } from 'pdfjs-dist/types/src/display/api'
  export default class PdfExtractor extends Vue {
    private extractedText = ''
    onChange(e: Event) {
      const input = e.target as HTMLInputElement
      const files = input.files
      if (files == null || files.length == 0) return
      const file = files[0]
      const reader = new FileReader()
      reader.onload = async () => {
        const arrayBuffer = reader.result as ArrayBuffer
        pdfjsLib.GlobalWorkerOptions.workerSrc =
          'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.7.107/pdf.worker.min.js'
        const loadingTask = pdfjsLib.getDocument(arrayBuffer)
        const pdf = await loadingTask.promise

        const totalPage = pdf._pdfInfo.numPages
        for (let pageNum = 1; pageNum <= totalPage; pageNum++) {
          const page = await pdf.getPage(pageNum)
          const data = await page.getTextContent()
          const pageItemArray = data.items
          for (let item of pageItemArray) {
            if ('str' in item) {
              const text = item as TextItem
              this.extractedText += text.str
            }
          }
        }
      }
      reader.readAsArrayBuffer(file)
    }
  }
</script>

<template>
  <div class="container">
    PDF Extractor
    <div class="fileInput">
      <input type="file" @change="onChange" />
    </div>
    <div class="resultCon">
      extract result
      <div class="result">
        {{ extractedText }}
      </div>
    </div>
  </div>
</template>

<style scoped></style>
