# 🎥 Case Study: Video Streaming (e.g. YouTube, Netflix)

## 📌 What is it?

A system that lets users **upload** video content and lets millions of others **stream** it smoothly, adapting to each viewer's network speed and device.

## 🤔 Requirements

### Functional

- Upload video content.
- Process/transcode video into multiple qualities/formats.
- Stream video to users with minimal buffering.

### Non-Functional

- **Massive read/bandwidth scale** — video is the heaviest content type on the internet.
- **Global low latency** — users everywhere need fast start times.
- **Adaptive playback** — smooth experience across varying network conditions (3G to fiber).
- Storage costs are enormous — must be efficient.

## 🧠 Core Design Question 1: How do you handle different devices/network speeds?

**Problem**: A single fixed-quality video file either buffers constantly on slow connections or looks poor on fast connections/large screens.

**Solution: Adaptive Bitrate Streaming (ABR)**

- The same video is **encoded into multiple resolutions/bitrates** (e.g. 240p, 480p, 720p, 1080p, 4K).
- Each version is split into small chunks (e.g. 2–10 seconds each).
- The video player **automatically switches quality** between chunks based on current network speed — no need to restart playback.

### 🖼 Adaptive Streaming Flow

```
                  ┌─────────────────────────┐
                  │  Original Uploaded Video  │
                  └────────────┬────────────┘
                                ▼
                     ┌────────────────────┐
                     │  Transcoding Service │
                     └────────────────────┘
             ┌─────────────┼─────────────┬─────────────┐
             ▼             ▼             ▼             ▼
         240p chunks   480p chunks   720p chunks   1080p chunks
             │             │             │             │
             └─────────────┴──────┬──────┴─────────────┘
                                   ▼
                          ┌────────────────┐
                          │  CDN (Edge)     │
                          └────────────────┘
                                   │
                                   ▼
                     ┌───────────────────────┐
                     │  Client Video Player    │
                     │ switches quality per    │
                     │ chunk based on network  │
                     └───────────────────────┘
```

Common protocols implementing this: **HLS** (HTTP Live Streaming, Apple) and **DASH** (Dynamic Adaptive Streaming over HTTP).

## 🧠 Core Design Question 2: How do you handle the upload → playback pipeline?

```
1. User uploads raw video file → stored in Object Storage (e.g. S3)
2. Upload triggers an async Transcoding Pipeline:
     - Splits into chunks
     - Encodes into multiple resolutions/bitrates
     - Generates thumbnails
3. Transcoded chunks stored in Object Storage
4. Metadata (video ID, available qualities, duration) saved to DB
5. Chunks distributed/cached across CDN edge servers globally
6. Video becomes available for streaming
```

This is inherently an **asynchronous, event-driven pipeline** — a user doesn't wait for full transcoding to complete synchronously; they get a "processing" status until it's ready (see `03_Event_Driven_Architecture.md`).

## 📊 Storage & Delivery Strategy

| Component                              | Technology                      | Why                                                                                                                       |
| -------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Raw + transcoded video files** | Object Storage (S3/GCS)         | Cheap, durable, built for large binary blobs                                                                              |
| **Video metadata**               | SQL/NoSQL DB                    | Title, description, video ID → chunk manifest mapping                                                                    |
| **Delivery to users**            | CDN (edge caching)              | Video must be served from**close to the user** — bandwidth-heavy content over long distances is slow and expensive |
| **Transcoding jobs**             | Distributed worker pool / queue | CPU-intensive, parallelizable across many chunks                                                                          |

## ⚡ Scaling Considerations

- **CDN is non-negotiable** here — video files are large, and without edge caching, origin servers and cross-continental bandwidth would be overwhelmed instantly (see `01_CDN.md`).
- **Popular videos** get cached at nearly every edge location (hot content); rarely-watched videos may only live in central/object storage, fetched on demand.
- **Transcoding is parallelizable**: split video into segments and transcode each segment concurrently across a worker pool rather than one long serial job.
- **Progressive availability**: lower resolutions (240p/480p) can be ready and available for viewing before higher resolutions finish transcoding, improving perceived speed.

## 🚨 Common Mistakes in Interviews

- ❌ Not mentioning adaptive bitrate streaming — proposing "just serve the video file" ignores real playback conditions.
- ❌ Forgetting the transcoding pipeline is **asynchronous** — treating upload as instantly ready to stream.
- ❌ Underestimating the central role of CDN — this is the single most critical component for this system specifically.
- ❌ Not considering storage cost tradeoffs (keeping every resolution forever vs re-transcoding on demand for rarely watched videos).

## 💡 Key Takeaways

- **Adaptive Bitrate Streaming** (HLS/DASH) is the standard solution for variable network conditions.
- Upload → Transcode → Store → CDN-distribute is an async, event-driven pipeline.
- CDN is the backbone of the entire delivery system — arguably the single most important infrastructure piece here.
- Object storage + metadata DB separation mirrors the general pattern: **big blobs in object storage, small structured data in a DB.**

## 🎤 Interview Questions

1. What is Adaptive Bitrate Streaming and why is it necessary for video platforms?
2. Why is the upload-to-playback pipeline designed as asynchronous rather than synchronous?
3. Why is a CDN especially critical for a video streaming system compared to, say, a typical web app?
4. How would you handle a hugely popular ("viral") video versus a rarely-watched one, from a storage/caching perspective?

## 📝 30-second Revision Cheat Sheet

- Adaptive Bitrate Streaming (HLS/DASH) = multiple resolutions, player auto-switches per chunk based on network speed.
- Pipeline: Upload → async Transcode → Object Storage → CDN → Playback.
- CDN is the backbone — video is the most bandwidth-heavy content type.
- Separate large blobs (object storage) from structured metadata (DB).
