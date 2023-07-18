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
      header="Cypress Test Reports"
    >
      <a-list-item v-for="r in reports" :key="r.name">
        <a :href="r.url" target="_blank">
          {{ r.name }}
        </a>
      </a-list-item>
    </a-list>
  </main>
</template>
