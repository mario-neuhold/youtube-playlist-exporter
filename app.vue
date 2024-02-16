<script lang="ts" setup>
import * as XLSX from 'xlsx'

const toast = useToast()

const config = useRuntimeConfig()

const accepted = useCookie('accepted', { default: () => false })
const api = 'https://youtube.googleapis.com/youtube/v3/'
const playlist = ref('')

const videos = ref([])

const fetchVideos = async () => {
  if (playlist.value === '') {
    toast.add({title: 'Please enter a playlist ID'})
    return
  }
  await $fetch(`${api}playlistItems`, {
    query: {
      part: 'contentDetails',
      maxResults: 50,
      playlistId: playlist.value,
      key: config.public.youtubeApiKey
    },
    onResponse({ response }) {
      if (response.status === 404) {
        toast.add({title: 'Playlist not found, check ID and that it is public or unlisted.'})
        return
      }
      if (response._data && response._data.items.length === 0) {
        toast.add({title: 'No videos found in playlist'})
        return
      }
      getVideoDetails(response._data.items.map((item: any) => {
        return item.contentDetails.videoId
      }))
    },
  })

}

const getVideoDetails = async (videoIds: any[]) => {
  await $fetch(`${api}videos`, {
    query: {
      part: 'snippet',
      id: videoIds.join(','),
      key: config.public.youtubeApiKey
    },
    onResponse({ response }) {
      videos.value = response._data.items.map((item: any) => {
        return {
          ID: item.id,
          Title: item.snippet.title,
          Channel: item.snippet.channelTitle,
          ChannelId: item.snippet.channelId,
          Thumbnail: item.snippet.thumbnails.default.url,
          URL: `https://www.youtube.com/watch?v=${item.id}`,
        }
      })
    }
  })

}

// download the data as an Excel file
const downloadXlsx = () => {

  if (videos.value.length === 0) {
    toast.add({title: 'No videos to download'})
    return
  }

  const worksheet = XLSX.utils.json_to_sheet(videos.value)
  
  // get the max lengths of each colum as array of objects { wch: number }
  const columnLengths = videos.value.reduce((acc, row) => {
    Object.keys(row).forEach((key, i) => {
      const l = row[key].toString().length
      if (!acc[i] || acc[i].wch < l) {
        acc[i] = { wch: l }
      }
    })
    return acc
  }, [])

  worksheet['!cols'] = columnLengths

  const workbook = XLSX.utils.book_new()

  XLSX.utils.book_append_sheet(workbook, worksheet, 'Videos')

  // save the file named after the playlist
  XLSX.writeFile(workbook, `playlist-${playlist.value}.xlsx`)

}


</script>
<template>
  <UContainer>
    <UCard class="mt-10">
      <template #header>
        <div class="flex justify-between">
          <h1 class="text-2xl font-bold">YouTube Playlist Export</h1>
          <ColorScheme>
            <USelect
              v-model="$colorMode.preference"
              :options="['system', 'light', 'dark']"
            />
          </ColorScheme>
        </div>
      </template>
      <p class="mb-8">
        This is a helper tool to fetch data from a YouTube playlist and download
        it as an Excel file. Everything happens in your browser and none of your
        data gets sent anywhere other than to Google for accessing the YouTube
        API.
      </p>

      <p class="mb-4">Work-in-progress! Current status:</p>
      <ul class="mb-2 list-disc ml-5">
        <li>✅ Get basic video info</li>
        <li>✅ Download as Excel</li>
        <li>🚧 Limited to 50 videos per playlist</li>
        <li>🚧 Missing data like views and likes</li>
      </ul>

      <div class="my-10"></div>
      <p class="mb-4">Enter the Playlist ID (example PLHbj3Gti2ieOqEtroiN4JuyyMs6fVvtAt)</p>
      <UInput v-model="playlist" />
      <div class="my-10"></div>
      <UButton
        icon="i-heroicons-arrows-right-left"
        @click="fetchVideos"
      >Load videos</UButton>
      <div class="my-10"></div>
      <UButton
        v-if="videos.length > 0"
        icon="i-heroicons-cloud-arrow-down"
        @click="downloadXlsx"
      >Download as XLSX</UButton>
      <div class="my-10"></div>
      <UTable
        v-if="videos.length > 0"
        :rows="videos"
        :columns="[
          { key: 'ID', label: 'ID' },
          { key: 'Title', label: 'Title' },
          { key: 'Channel', label: 'Channel' },
          { key: 'ChannelId', label: 'Channel ID' },
          { key: 'Thumbnail', label: 'Thumbnail' },
          { key: 'URL', label: 'URL' },
        ]"
      >
        <template #Thumbnail-data="{ row }">
          <img
            :src="row.Thumbnail"
            alt="thumbnail"
            class="w-20"
          />
        </template>
        <template #URL-data="{ row }">
          <a
            :href="row.URL"
            target="_blank"
          >{{ row.URL }}</a>
        </template>
      </UTable>
    </UCard>
    <UNotifications />
  </UContainer>
</template>
