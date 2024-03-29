# Dr. Slump Brasil Downloader
This is a project to download episodes from the [Dr. Slump Brasil site](https://drslumpbrasil.blogspot.com) with PT-BR fansub. If you like the episodes, you can donate to the site owner, more about it on the Dr. Slump Brasil site.

The script will get all entries (posts) from the site, download them, rename, and change the files' ownership (user and group). It also uses a SQLite database to keep track of already downloaded entries, so it can skip them the next time the script runs. Because of that, you can create a [Cron](https://en.wikipedia.org/wiki/Cron) job to execute the project daily and download new entries from the site as they're released.

# How to run:
The project uses environment variables to define some configs:
```
# Where to download the episodes 
DOWNLOAD_FOLDER=/abs/path/to/folder

# SQLite database to be used to keep track of already downloaded episodes
DB_PATH=/abs/path/to/file.db

# Filter to decide what types of episodes to download. Can be: 80show, 90show, specials, movies, all
DOWNLOAD_FILTER=all

# Set the user ID and group ID of the downloaded files
FILES_UID=1000
FILES_GID=1000

# (Optional) You can enable notifications to a Ntfy (https://ntfy.sh) topic by exporting the following environment variables.
# They'll notify when an episode is downloaded or an error occurs in the execution:
NTFY_ADDRESS=https://sub.domain.com
NTFY_TOPIC=topic_name
NTFY_TOKEN=token

# (Optional) Set the MEGAnz account to use when downloading files
export MEGA_EMAIL=
export MEGA_PASSWORD=
```
## Run using Docker
1. Build the Docker image:
```sh
docker build -t dr-slump-brasil .
```
2. Run:
```sh
docker run --name dr-slump-brasil -v ./downloads:/downloads -v ./data:/data -e DOWNLOAD_FILTER=filter -e FILES_UID=1000 -e FILES_GID=1000 dr-slump-brasil

# (Optional) Run with Ntfy integration:
docker run --name dr-slump-brasil -v ./downloads:/downloads -v ./data:/data -e DOWNLOAD_FILTER=filter -e NTFY_ADDRESS=https://sub.domain.com -e NTFY_TOPIC=topic_name -e NTFY_TOKEN=token dr-slump-brasil

# (Optional) Run with a Mega account
docker run --name dr-slump-brasil -v ./downloads:/downloads -v ./data:/data -e DOWNLOAD_FILTER=filter -e MEGA_EMAIL=email -e MEGA_PASSWORD=password dr-slump-brasil
```

## Run manually
### Requirements
- Install [MEGAcmd](https://github.com/meganz/MEGAcmd) CLIs, they'll be used because some episodes are available on [MEGA.nz](https://mega.nz).
- Install the Python dependencies:
```sh
pip install -r requirements.txt
```
1. Start the mega-cmd-server:
```
mega-cmd-server
```
2. Export the environment variables needed:
```sh
export DOWNLOAD_FOLDER=/abs/path/to/folder
export DB_PATH=/abs/path/to/file.db
export DOWNLOAD_FILTER=filter
export FILES_UID=1000
export FILES_GID=1000

# Optional
export NTFY_ADDRESS=https://sub.domain.com
export NTFY_TOPIC=topic_name
export NTFY_TOKEN=token

# Optional
export MEGA_EMAIL=user@email.com
export MEGA_PASSWORD=pass
```
3. Execute the script:
```sh
python3 main.py
```

# Limitations
## Download from MEGAnz
Mega has a bandwidth quota limitation, where you can download a certain amount of GBs until you reach the bandwidth quota limit for your IP address, after this, you need to wait to download again. The project will download the episodes anyway, but when you reach the quota limit, the download will pause and resume when Mega gives you more quota or reset your quota usage (it can take hours!).
- Most (*if not all*) of the specials, movies, and the 80's show episodes are uploaded to Mega, so this is a huge slowdown for these downloads.
- Most (*if not all*) of the 90's show files are uploaded to Google Drive or OneDrive, so they don't have this issue.
## Download from OneDrive
Some files are available on OneDrive, but the project won't directly use the OneDrive URL to download the file (I don't know how to do it simply and I spent too much time testing how to). Instead, I accessed each link to get the "real" download URL and created a map variable between the OneDrive URL available on the site and the "real" download URL, **manually**. I did this because currently, only a few episodes are in OneDrive (episodes 62-74 of the 90's show), so it wasn't so much trouble.
- The map is on the file `src/download.py`, with the variable name `ONEDRIVE_MAP`.
