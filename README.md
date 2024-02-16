# YouTube Playlist Exporter

A simple web application to export YouTube playlists to an Excel file.

## Status: 🚧 In Development

- ✅ Get basic video info
- ✅ Download as Excel
- 🚧 Limited to 50 videos per playlist
- 🚧 Missing data like views and likes
- ...

## Technologies

- [Nuxt 3](https://nuxt.com/)
- [Nuxt UI](https://ui.nuxt.com/)
- [SheetJS](https://sheetjs.com/)


## Local development

### Install the dependencies:

```bash
# npm
npm install

# pnpm
pnpm install

# yarn
yarn install

# bun
bun install
```

Get a YouTube Data API key and save it in a `.env` file as `NUXT_PUBLIC_YOUTUBE_API_KEY`:

```bash
NUXT_PUBLIC_YOUTUBE_API_KEY=your-api-key
```


### Start the development server on `http://localhost:3000`:

```bash
# npm
npm run dev

# pnpm
pnpm run dev

# yarn
yarn dev

# bun
bun run dev
```

