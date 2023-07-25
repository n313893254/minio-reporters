<script lang="ts">
import { Minio } from 'minio-js'

const MINIO_BUCKET_NAME = 'cypress-test-report'
const MINIO_DIR = 'cypress/results/'

export default {
  data() {
    return {
      reports: [] as Array<{ name: string; url: string }>,
    }
  },

  async created() {
    const minioClient = new Minio.Client({
      endPoint: 'minio.provo.rancherlabs.com',
      port: 31524,
      useSSL: true
    })

    minioClient.setRequestOptions({ rejectUnauthorized: false })

    const stream = minioClient.listObjectsV2(MINIO_BUCKET_NAME, MINIO_DIR, false, '')
    const objectList: Array<{ prefix: string }> = await this.streamToList(stream)

    this.reports = objectList.map(({ prefix }) => this.getPathToUrl(prefix))
  },

  methods: {
    streamToList(stream: any): Promise<any[]> {
      return new Promise((resolve, reject) => {
        const list: any[] = []
        stream.on('data', (obj: any) => list.push(obj))
        stream.on('error', reject)
        stream.on('end', () => resolve(list))
      })
    },

    getPathToUrl(path: string) {
      const folder = path.split('/').filter(Boolean).pop() || ''

      return {
        name: folder,
        url: `./${MINIO_DIR}${folder}/index.html`
      }
    }
  }
}
</script>

<template>
  <main>
    <a-list
      :pagination="{
        pageSize: 10,
      }"
      :data-source="reports.sort((a, b) => b.name.localeCompare(a.name))"
    >
      <template #header>
        <div class="banner-graphic">
          <div class="graphic">
            <img src="./assets/banner.svg"/>
          </div>
          <h1 class="title">
            Cypress Test Reports
          </h1>
        </div>
      </template>
      <template #renderItem="{ item: report }">
        <a-list-item :key="report.name">
          <a :href="report.url" target="_blank">
            {{ report.name }}
          </a>
        </a-list-item>
      </template>
    </a-list>
  </main>
</template>

<style lang="scss" scoped>
.banner-graphic {
    position: relative;
}

.banner-graphic .graphic {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

h1 {
  font-size: 24px;
  font-style: normal;
  font-weight: 400;
  margin: 0 0 10px;
}

.title {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  text-align: center;
  top: 0;
  height: 100%;
  width: 100%;
}
</style>
