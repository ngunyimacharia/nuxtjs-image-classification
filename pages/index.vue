<template>
  <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
    <div class="shadow m-10 text-center p-5">
      <h1 class="m-5 text-lg">Select image to tag</h1>
      <p class="m-5 text-sm">
        This utility will automatically tag the image based on it's contents
      </p>
      <input
        type="file"
        name="image"
        id="image"
        required
        accept="image/*"
        @change="handle"
      />
      <div class="text-sm text-yellow-600" v-if="tagging">Tagging...</div>
      <div class="text-sm text-yellow-600" v-if="uploading">Uploading...</div>
      <div class="text-sm text-red-500" v-if="error">{{ error }}</div>
    </div>

    <section class="mt-8 pb-16">
      <ul
        role="list"
        class="
          grid grid-cols-2
          gap-x-4 gap-y-8
          sm:grid-cols-3
          sm:gap-x-6
          md:grid-cols-4
          lg:grid-cols-3
          xl:grid-cols-4
          xl:gap-x-8
        "
      >
        <li v-for="(image, idx) in images" :key="idx" class="relative">
          <div
            class="
              focus-within:ring-2
              focus-within:ring-offset-2
              focus-within:ring-offset-gray-100
              focus-within:ring-indigo-500
              group
              block
              w-full
              aspect-w-10 aspect-h-7
              rounded-lg
              bg-gray-100
              overflow-hidden
            "
          >
            <img
              :src="`${image.secure_url}?ap=em`"
              :alt="`Image ${image.public_id}`"
              class="group-hover:opacity-75 object-cover pointer-events-none"
            />
          </div>
          <p
            class="
              mt-2
              block
              text-sm
              font-medium
              text-gray-900
              truncate
              pointer-events-none
            "
          >
            {{ image.public_id }}
          </p>
          <p
            class="block text-sm font-medium text-gray-500 pointer-events-none"
          >
            {{ image.bytes }} bytes
          </p>
          <div>
            <span
              class="
                mr-2
                inline-flex
                items-center
                px-2.5
                py-0.5
                rounded-full
                text-xs
                font-medium
                bg-gray-100
                text-gray-800
              "
              v-for="(tag, idx) in image.tags"
              :key="idx"
            >
              {{ tag }}
            </span>
          </div>
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tagging: false,
      uploading: false,
      error: false,
      images: [],
    };
  },
  methods: {
    async handle(e) {
      let file = e.target.files[0];

      const fileData = await this.readData(file);

      this.getTags(fileData);
    },

    async getTags(fileData) {
      this.tagging = true;

      const data = {
        requests: [
          {
            image: { content: fileData.split(",")[1] },
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
        this.tagging = false;
        this.error = resp.responses[0].error.message;
        return;
      }

      this.error = null;

      const tags = resp.responses[0].labelAnnotations.map(
        (label) => label.description
      );

      this.upload(fileData, tags);

      this.tagging = false;
    },

    async upload(fileData, tags) {
      this.uploading = true;

      const uploadResp = await this.$cloudinary.upload(fileData, {
        upload_preset: "default-preset",
        folder: "nuxtjs-image-classification/tagged",
        tags,
      });

      this.images.push(uploadResp);

      this.uploading = false;
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
