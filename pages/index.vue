<template>
  <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
    <input
      type="file"
      name="image"
      id="image"
      required
      class="
        focus:ring-indigo-500
        focus:border-indigo-500
        flex-1
        block
        w-full
        rounded-none rounded-r-md
        sm:text-sm
        border-gray-300
      "
      @change="upload"
    />
    <div v-if="uploading">Uploading...</div>
    <div v-if="error">{{ error }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      uploading: false,
      error: false,
      images: [
        {
          asset_id: "e27378401b77573969debb652b0b9303",
          public_id: "nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug",
          version: 1630413200,
          version_id: "dcc81c459b7cb4f768101689a2b8aec2",
          signature: "6a072342985e8b404cc84c9ffff951a25cc3cef2",
          width: 1280,
          height: 960,
          format: "jpg",
          resource_type: "image",
          created_at: "2021-08-31T12:33:20Z",
          tags: [],
          bytes: 150613,
          type: "upload",
          etag: "c89f89f3bf2b657f56de8268b61720b1",
          placeholder: false,
          url: "http://res.cloudinary.com/hackit-africa/image/upload/v1630413200/nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug.jpg",
          secure_url:
            "https://res.cloudinary.com/hackit-africa/image/upload/v1630413200/nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug.jpg",
          access_mode: "public",
        },
        {
          asset_id: "e27378401b77573969debb652b0b9303",
          public_id: "nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug",
          version: 1630413200,
          version_id: "dcc81c459b7cb4f768101689a2b8aec2",
          signature: "6a072342985e8b404cc84c9ffff951a25cc3cef2",
          width: 1280,
          height: 960,
          format: "jpg",
          resource_type: "image",
          created_at: "2021-08-31T12:33:20Z",
          tags: [],
          bytes: 150613,
          type: "upload",
          etag: "c89f89f3bf2b657f56de8268b61720b1",
          placeholder: false,
          url: "http://res.cloudinary.com/hackit-africa/image/upload/v1630413200/nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug.jpg",
          secure_url:
            "https://res.cloudinary.com/hackit-africa/image/upload/v1630413200/nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug.jpg",
          access_mode: "public",
        },
        {
          asset_id: "e27378401b77573969debb652b0b9303",
          public_id: "nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug",
          version: 1630413200,
          version_id: "dcc81c459b7cb4f768101689a2b8aec2",
          signature: "6a072342985e8b404cc84c9ffff951a25cc3cef2",
          width: 1280,
          height: 960,
          format: "jpg",
          resource_type: "image",
          created_at: "2021-08-31T12:33:20Z",
          tags: [],
          bytes: 150613,
          type: "upload",
          etag: "c89f89f3bf2b657f56de8268b61720b1",
          placeholder: false,
          url: "http://res.cloudinary.com/hackit-africa/image/upload/v1630413200/nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug.jpg",
          secure_url:
            "https://res.cloudinary.com/hackit-africa/image/upload/v1630413200/nuxtjs-image-classification/rs3ak4vdkdlp9ylsvnug.jpg",
          access_mode: "public",
        },
      ],
    };
  },
  methods: {
    async upload(e) {
      this.uploading = true;
      // Instructor video file
      let file = e.target.files[0];
      /* Read data */
      const data = await this.readData(file);
      /* upload the converted data */
      const resp = await this.$cloudinary.upload(data, {
        upload_preset: "default-preset",
        folder: "nuxtjs-image-classification",
      });
      this.label(resp, data);
      this.uploading = false;
    },
    async label(image, imageData) {
      const url = this.$cloudinary.image.url(image.public_id);

      const data = {
        requests: [
          {
            image: {
              source: {
                imageUri: url,
              },
            },
            features: [
              {
                maxResults: 5,
                type: "LABEL_DETECTION",
              },
            ],
          },
        ],
      };

      const resp = await this.$axios.$post(
        `https://vision.googleapis.com/v1/images:annotate?key=${process.env.NUXT_ENV_GOOGLE_VISION_API_KEY}`,
        data
      );

      if (resp.responses[0].error) {
        this.error = resp.responses[0].error.message;
        return;
      }

      this.error = null;

      const tags = resp.responses[0].labelAnnotations.map(
        (label) => label.description
      );

      const uploadResp = await this.$cloudinary.upload(imageData, {
        upload_preset: "default-preset",
        folder: "nuxtjs-image-classification/tagged",
        tags,
      });

      this.images.push(uploadResp);
    },
    readData(f) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onloadend = () => resolve(reader.result);
        reader.readAsDataURL(f);
      });
    },
  },
};
</script>
