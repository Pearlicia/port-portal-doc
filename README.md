# Guide: YouTube Playlist & Videos Catalog in Port

## Introduction
In this guide, I will demonstrate how Port can efficiently manage and visualize data from a YouTube playlist, helping you get insights at a glance. Port is a robust platform for modeling and cataloging diverse data types, and this guide will showcase its capabilities for managing video data.

### Why Use Port?
Port offers a unique approach to data management:
1. **Flexible Data Modeling**: With custom blueprints and properties, you can create a tailored structure to organize various types of data.
2. **Data Ingestion with Automation**: Using GitHub workflows, you can automate the data import process, making it simple to keep your data updated.
3. **Powerful Visualizations**: Port’s visualization tools help you identify patterns, trends, and metrics, making data-driven decisions easier.

## Prerequisites 
1. Port Account. Go to [port website](https://app.getport.io) to create an account 
2. GitHub Account. 
3. YouTube data API V3.
   * 1. Log in to Google Developers Console.
    You can log in to Google Developers Console using your Google account. If you don’t yet have one, you’ll need to    create one.

     2. Create a new project.
     Once you log in, you’ll automatically be taken to an empty dashboard. On the upper right-hand corner, click Create      Project.

Google Developers Console dashboard 

Image Source

You’ll be taken to a screen where you can add a project name, select your organization, and select a location (URL).

You can create a project without an organization. If you don’t have an organization, you won’t be prompted to select a location.

New project form in Google Developer Console

Image Source

Once you create a project, everything you do from here onwards, including creating the API key, will apply to that specific project. If you have multiple projects going at the same time, ensure that the correct one is selected by double-checking that the project is listed on the top navigation bar.

3. On the new project dashboard, click Explore & Enable APIs.
After creating a project, you’ll be taken to a brand new dashboard with different cards: Project Info, Resources, and so on.

Navigate to the Getting Started card and click on Explore & Enable APIs.

New project dashboard in Google Developers Console

Image Source

Alternatively, on the sidebar, navigate to APIs & Services > Library.

4. In the library, navigate to YouTube Data API v3 under YouTube APIs.
Once you’ve reached the library, you’ll see a page divided into sections. Navigate to the section titled YouTube APIs and select YouTube Data API v3.

Google's API library, where the YouTube API is located

Image Source

Alternatively, you can search for the YouTube Data API on the search bar.

5. Enable the API.
After you arrive at the YouTube Data API page, click a blue button with the word Enable.

YouTube API page on the Google Developers Console's API library

Image Source

6. Create a credential.
After clicking Enable, you’ll be taken to an overview page.

Dashboard for the YouTube Data API, where you can then create a credential

Image Source

On the top right corner, click Create Credentials.

On the Credentials window, select YouTube Data API v3 for the first blank space, Web server (e.g. node js. Tomcat) for the second, and check the Public data box on the third prompt.

Credential configurations for creating a YouTube API key

Image Source

Right below that, click on the blue button titled What credentials do I need?

After that, your API key will automatically load.

YouTube API created after clicking "What credentials do I need?" 

Image Source

Click Done.

7. A screen will appear with the API key.
Finished! You’ve got your API key. It’s saved in your credentials for easy retrieval.

YouTube API key created and listed on the API & Services dashboard in Google Developers Console

Image Source


## Step 1: Model Data in Port

The first step is to define a model for the YouTube playlist and videos in Port. This involves creating *blueprints* for the playlist and each video, defining their properties, and establishing relationships.

### 1.1. Create Blueprints
- **Playlist Blueprint**: This represents the playlist as a whole.
  - **Properties**:
    - **Playlist ID** (Unique ID)
    - **Title** (String)
    - **Description** (String)
    - **Published Date** (Date)
    - **Total Videos** (Number)
- **Video Blueprint**: Each video is an item within the playlist.
  - **Properties**:
    - **Video ID** (Unique ID)
    - **Title** (String)
    - **Description** (String)
    - **Published Date** (Date)
    - **Duration** (Time)
    - **View Count** (Number)

### 1.2. Define Relationships
- Link each video to the playlist using a *one-to-many relationship*. This means one playlist blueprint can contain multiple video blueprints.

**Example JSON for Playlist Blueprint**:
```json
{
  "blueprint": "Playlist",
  "properties": {
    "Playlist ID": "string",
    "Title": "string",
    "Description": "string",
    "Published Date": "date",
    "Total Videos": "number"
  }
}
```

**Example JSON for Video Blueprint**:
```json
{
  "blueprint": "Video",
  "properties": {
    "Video ID": "string",
    "Title": "string",
    "Description": "string",
    "Published Date": "date",
    "Duration": "time",
    "View Count": "number"
  },
  "relationships": {
    "belongs_to": "Playlist"
  }
}
```

## Step 2: Set Up GitHub Workflow

This step involves creating a workflow to fetch data from YouTube and ingest it into Port.

### 2.1 Fetch Data from YouTube
To fetch data, you’ll need to use the YouTube Data API. Here’s how to set up a basic request:

1. **Get API Key**: Register for an API key on the [Google Developer Console](https://console.developers.google.com/).
2. **Fetch Playlist Data**:
   - Use the endpoint `https://www.googleapis.com/youtube/v3/playlists`.
   - Set parameters: `id` (playlist ID), `part` (snippet, contentDetails).

**Example API Call**:
```bash
curl -X GET "https://www.googleapis.com/youtube/v3/playlists?id=YOUR_PLAYLIST_ID&part=snippet,contentDetails&key=YOUR_API_KEY"
```

3. **Fetch Video Data**:
   - Use the endpoint `https://www.googleapis.com/youtube/v3/playlistItems`.
   - Set parameters: `playlistId` (playlist ID), `part` (snippet, contentDetails).

**Example API Call**:
```bash
curl -X GET "https://www.googleapis.com/youtube/v3/playlistItems?playlistId=YOUR_PLAYLIST_ID&part=snippet,contentDetails&key=YOUR_API_KEY"
```

### 2.2 Ingest Data into Port
Once you’ve fetched the data, use Port’s API to upload it.

1. **Format Data in JSON**: Ensure the data matches the blueprint properties defined earlier.
2. **Automate with GitHub Actions**:
   - Write a GitHub Action that periodically runs the fetch and ingest commands.
   - **Example Workflow**:
     ```yaml
     name: Ingest YouTube Data to Port

     on:
       schedule:
         - cron: '0 0 * * *' # Runs daily

     jobs:
       fetch-and-ingest:
         runs-on: ubuntu-latest
         steps:
           - name: Fetch YouTube Data
             run: |
               # Fetch data and save to file
               curl -X GET "https://www.googleapis.com/youtube/v3/playlistItems?playlistId=YOUR_PLAYLIST_ID&part=snippet,contentDetails&key=YOUR_API_KEY" > data.json

           - name: Ingest to Port
             run: |
               # Use Port's API to upload data
               curl -X POST "https://api.getport.io/v1/ingest" -H "Authorization: Bearer YOUR_PORT_TOKEN" -H "Content-Type: application/json" -d @data.json
     ```

## Step 3: Visualize the Data in Port

Port offers tools to visualize data, allowing you to track key metrics like view counts, publishing trends, and playlist growth. Here are two examples of valuable visualizations:

### Visualization 1: Video Publish Trends
- **Objective**: Show the number of videos published over time.
- **Usage**: Identify how frequently new videos are added to the playlist.

### Visualization 2: View Count Distribution
- **Objective**: Display the view count of each video to identify popular content.
- **Usage**: Understand which topics or types of content resonate most with viewers.

After creating these visualizations, add screenshots or explanations to your guide to help users interpret the data.

## Conclusion
With this guide, you now have a blueprint to create a YouTube playlist catalog in Port. By automating data ingestion and using visualizations, Port provides a powerful, centralized solution to manage and understand video data at scale. This approach will save time, ensure data consistency, and provide actionable insights for your audience.

