<template>
  <div>
    <div class="flex items-center justify-between">
      <h1 class="font-bold text-2xl">eBooks - Catálogo</h1>
    </div>

    <div class="mt-6">
      <div class="flex justify-center flex-wrap gap-10">
        <div
          v-for="ebook in ebooks"
          :key="ebook.id"
          @click="showEbook(ebook.pdf)"
          class="
            w-60
            shadow-2xl
            border-b-4 border-transparent
            cursor-pointer
            hover:border-indigo-500
          "
        >
          <nuxt-img :src="ebook.img" />
        </div>
      </div>
    </div>
    <vs-dialog scroll overflow-hidden auto-width v-model="active">
      <template #header>
        <div class="flex justify-between gap-10 p-4">
          <div class="flex items-center space-x-3">
            <div class="text-lg font-bold">Assinatura do eBook:</div>
            <div>
              <AppFormInput v-model="signature" />
            </div>
          </div>
          <div>
            <AppButton @click="modifyPdf"> Download </AppButton>
          </div>
        </div>
      </template>
      <div class="flex justify-center">
        <pdf-preview :url="srcPdf"></pdf-preview>
      </div>
    </vs-dialog>
  </div>
</template>

<script>
import AppButton from "~/components/Ui/AppButton";
import AppFormInput from "~/components/Ui/AppFormInput";
import { degrees, PDFDocument, rgb, StandardFonts } from "pdf-lib";
import downloadjs from "downloadjs";
import pdfPreview from "~/components/PdfPreview.vue";

export default {
  name: "eBooksPage",
  components: {
    AppButton,
    AppFormInput,
    pdfPreview,
  },
  data() {
    return {
      active: false,
      input1: "",
      input2: "",
      checkbox1: false,

      signature: "",
      srcPdf: "",
      ebooks: [
        {
          id: 1,
          img: "/img/ebook1.png",
          pdf: "/ebooks/ebook1.pdf",
        },
        {
          id: 2,
          img: "/img/ebook2.png",
          pdf: "/ebooks/ebook2.pdf",
        },
        {
          id: 3,
          img: "/img/ebook3.png",
          pdf: "/ebooks/ebook3.pdf",
        },
        {
          id: 4,
          img: "/img/ebook4.png",
          pdf: "/ebooks/ebook4.pdf",
        },
        {
          id: 5,
          img: "/img/ebook5.png",
          pdf: "/ebooks/ebook5.pdf",
        },
        {
          id: 6,
          img: "/img/ebook6.png",
          pdf: "/ebooks/ebook6.pdf",
        },
      ],
    };
  },
  methods: {
    showEbook(pdf) {
      this.active = true;
      this.srcPdf = pdf;
    },
    async modifyPdf() {
      if (!this.signature) {
        return;
      }

      const url = this.srcPdf;
      const existingPdfBytes = await fetch(url).then((res) =>
        res.arrayBuffer()
      );

      const pdfDoc = await PDFDocument.load(existingPdfBytes);
      const helveticaFont = await pdfDoc.embedFont(StandardFonts.Helvetica);

      const pages = pdfDoc.getPages();
      for (let p = 0; p < pages.length; p++) {
        pages[p].drawText(this.signature, {
          x: 15,
          y: 15,
          size: 20,
          font: helveticaFont,
          color: rgb(0.0, 0.14, 0.4),
          rotate: degrees(0),
        });
      }

      const pdfBytes = await pdfDoc.save();

      // Trigger the browser to download the PDF document
      downloadjs(
        pdfBytes,
        `ebook-for-${this.signature}.pdf`,
        "application/pdf"
      );
    },
  },
};
</script>
