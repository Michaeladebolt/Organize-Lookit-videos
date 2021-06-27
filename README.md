## Overview

One thing you may have noticed is that when you download the video data from Lookit, it downloads into a single file making it very difficult to know which file belongs to which participant. One soultion would be to look up the ID information included in each file name, but because the file names are so long and because there are so many files in the downloaded folder, this process would be very arduous. In addition, every time you wish to download *new* data, you would have to repeat this process from the beginning again. 

A solution to this issue is to use R to automatically move all your videos into sub-folders for each participant in your study! This is the purpose of this tutorial. 

This file organizes all the videos from all of the participants in the study. Specifically, this code creates a folder for each participant using their "child_hashed_id" assigned to them within Lookit. If the same participant participates in the study more than once, all of their study videos will be organized in the same folder. 

Before running the code below, you will need to create 2 folders entitled, "lookit_videos" and "response_overview". The lookit_videos folder will contain all of the video files you downloaded from Lookit in a single folder. It it important that you don't edit the file names in any way. The response_overview folder will contain the Your-Study-Name_all-responses-identifiable file.csv. 

This file can be downloaded from the Lookit site under the "All Responses" heading and the "Response overview" subsection. It doesn't matter if you select identifying information or not; it is important, however, that this file contains the child's hashed id ("child_hashed_id") and the response_uuid ("response_uuid").

First, you will need to load the `tidyverse` and `stringr` packages.


```r
library(stringr)
library(tidyverse)
```


Next, you will need to set various working directories in your script. The `lookitVideos` path will point to the folder that contains all of your video data downloaded from Lookit. In this example, "lookit_videos" is the name of the folder that contains all the videos from my study. 

You will also want to create a variable that contains the path to where your response data is located. In this example, the folder "response_overview_data" is the name of the folder that contains the "response-overview" data downloaded from Lookit. 


```r
# Change these working directory paths to match the location of where these folders/files are organized on your computer
lookitVideos <- "/Volumes/General/Backup/CodingProjects/Smiles_and_Masks_LookitStudy/lookit_videos/" # Location that contains all the videos downloaded
# from lookit
responseData <- "~/Box/Research/Smiles_and_Masks_LookitSudy_2020/data/response_overview_data/" # location where response data are stored
```


Now it's time to read in the relevant data that you downloaded from Lookit. 


```r
responseDat <-  read.csv(paste0(responseData, "Smiles-and-Masks_all-responses-identifiable.csv"), na.strings = " ")
```

In the next step, you will create a data frame with the unique subject ID information.


```r
setwd(lookitVideos) # set working directory to the location containing all the videos downloaded from Lookit

# Create a data frame containing the names of all the video files
allVideos <- data.frame(filename = list.files(path = lookitVideos, pattern = ".mp4"))

# Split up the name of the video file into different parts in order to extract the different pieces of information coneyed in the data file name
allVideos2 <- separate(data = allVideos, col = filename, sep = "_", into = c("X1","X2" ,"trial_type", "response_uuid"),remove = F)

# Merge the video file name information with the response data in order to get the child'd ID information
allVideos3 <- merge(allVideos2, responseDat, by.x = "response_uuid", by.y = "response__uuid")

# Lastly, create a list of all the unique child ID's from the allVideos3 data frame
sub_ids <- unique(allVideos3[,c("response_uuid","child__hashed_id")])  # create a list of the study IDs
```

Lastly, you will create a *new* folder to store all the organized videos for each Child ID. This folder will be created within the same folder in which all of your video files are stored. This can be changed by substituting another location for the "lookitVideos" path above. 


```r
newDir <- paste0(lookitVideos, "organized_lookit_videos_", Sys.Date()) # Create the new folder name with today's date
dir.create(newDir) # Create the new folder 

# Loop through unique subject IDs and put videos in their new homes
for (i in 1:nrow(sub_ids))
{
  filenames <- list.files(path = lookitVideos, pattern = sub_ids[i,1]) #list all the files for a specific subject
  newSubDir <- dir.create(paste0(newDir, "/", sub_ids[i,2])) # Create new subject-specific folder
  file.copy(from = filenames, to = paste0(newDir, "/", sub_ids[i,2])) # Copies all the files into the new homes
}# end for loop
```

That's it! Now you should have a new folder entitled "organized_lookit_videos" with the date appended that contains folders for each child that participated in your study. Within each folder for each child, you should see all of the videos for that one child *within* their own folder. 






