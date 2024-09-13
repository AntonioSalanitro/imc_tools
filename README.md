# imc_tools
Some useful tools to process IMC (Imaging Mass Cytometry) data from Hyperion.

## Overview

The pipeline is composed of several steps to facilitate the processing and analysis of IMC data, including image extraction, segmentation, and data analysis.

### Preliminary Data Check

Before proceeding with the automation, raw data undergo a preliminary check using software such as MCD Viewer and Napari to ensure their quality and integrity. After this initial inspection, the MCD file should be placed in the working directory within a sub-directory named “raw.” Additionally, the working directory must contain the `panel.csv` file, which is manually prepared to specify the markers to include in the analysis, guiding the process to ensure that only the markers of interest are processed. There should also be a sub-directory named “compensation,” where the necessary `.txt` files for compensation are located.

### Pipeline Components

1. **Image Extraction**: Filtering of images using Steinbock.
2. **Segmentation of Images**: Extracts image data from MCD files using Steinbock (https://bodenmillergroup.github.io/steinbock) and applies custom hot pixel filtering.
3. **IMC Data Analysis**: Data analysis leveraging the IMCDataAnalysis container (https://bodenmillergroup.github.io/IMCDataAnalysis/).
4. **Co-expression Analysis**: A Python script is included for the co-expression analysis of markers.

### Automation of the Analysis Pipeline

Automating the analysis pipeline has been a crucial step to optimize the workflow, improve the reproducibility of results, and ensure efficient data management and processing. The entire process has been automated using a combination of Bash and R scripts, allowing for consistent and uninterrupted execution of the various stages of the pipeline.

The automation process begins with image preprocessing and segmentation, carried out through a Bash script. This script is well-documented and includes an introductory section with essential information such as the script's purpose, author, creation date, and current version. The scripts are hosted in a dedicated Git repository, accessible via this link (LINK TO BE INSERTED). Users can clone the repository or download the scripts directly to their operating system for local execution.

The Bash scripts require two main arguments from the user: the main path to the Steinbock folders, containing all the sample folders to be processed, and the path to the `panel.tsv` file, which is crucial for segmentation with DeepCell as it contains information about the channels to use. Before proceeding, the script checks for the presence and validity of these files to prevent errors during execution. Preliminary checks are then performed to ensure that the provided paths are correct and that all necessary files are available.

To invoke the script, use the following command in the terminal:

```bash
bash script_name.sh /path/to/steinbock /path/to/panel.tsv
```

### Scripts and Commands

To extract images:

```bash
bash imc_extraction.sh -r ROOTFOLDER
```

where `ROOTFOLDER` is the folder containing the directory "samples."

To segment images:

```bash
bash imc_segmentation.sh -r ROOTFOLDER -p panel.tsv
```

where `ROOTFOLDER` is the folder containing the directory "samples," and `panel.tsv` is the file containing the markers.

### Using Docker for Image Processing

The automation of the preprocessing and segmentation of IMC images is carried out using Docker, which ensures a controlled and reproducible execution environment. The preprocessing includes operations such as "hot pixel" filtering. After preprocessing, the images are subjected to the DeepCell segmentation model integrated into Steinbock, which divides the images into regions corresponding to individual cells. The script then performs a series of measurements on the segmented cells, such as pixel intensity and morphological properties.

Once the job is complete, the script cleans up the working environment by removing unnecessary Docker containers and logs the end time of the execution process. This automation allows for the efficient processing of large amounts of data without manual intervention.

### Data Analysis with R

An R script is used for data analysis, configurable via an external configuration file (`args.txt`). This file must be placed in the correct directory within the Docker container, specifically `/home/rstudio/IMCDataAnalysis/`, to be accessible by the script. The `args.txt` file contains essential parameters for the analysis, such as the list of markers to use, channels to exclude, the working directory for saving results, the scale value for the arcsinh transformation of data, the decision on applying batch correction, and the use of Louvain clusters.

To run the script inside the Docker container, use the following command, which allows you to execute the script within the properly configured and running Docker container (note that the container must be started as explained in the previous section before executing this command):

```bash
docker exec -it $(docker ps -qf "ancestor=ghcr.io/bodenmillergroup/imcdataanalysis:latest") Rscript /home/rstudio/IMCDataAnalysis/script.R
```

### Explanation of the Docker Command

The `docker exec` command is used to execute an action within an already running Docker container. In the specified command, the `-it` options maintain the container's standard input open for interaction (`-i`) and allocate a pseudo-TTY terminal (`-t`), allowing readable output display.

The part `$(docker ps -qf "ancestor=ghcr.io/bodenmillergroup/imcdataanalysis:latest")` represents a command executed within a subshell, the result of which is passed as a parameter to `docker exec`. In detail, `docker ps` lists all active Docker containers; the `-q` option limits the output to container IDs, while `-f "ancestor=ghcr.io/bodenmillergroup/imcdataanalysis:latest"` filters only containers using the specified image (`ghcr.io/bodenmillergroup/imcdataanalysis:latest`). Thus, this part of the command identifies the ID of the container using that image.

The command `Rscript /home/rstudio/IMCDataAnalysis/script.R` is executed within the selected container. `Rscript` is a tool from the R suite that allows the execution of R scripts directly from the command line. The path `/home/rstudio/IMCDataAnalysis/script.R` indicates the location of the R script within the container to be executed.

Each parameter is checked at the beginning of the execution to ensure it has been correctly provided and is compatible with the analysis process, thus preventing errors during execution. This well-documented and methodical approach ensures that all samples are treated uniformly, significantly contributing to the reliability and effectiveness of scientific research.

### Additional Tools

- **IMC Data Analysis in R**: An R script is included to perform data analysis using the IMCDataAnalysis container. This script utilizes the container to perform various analyses, including clustering and batch correction.
- **Co-expression Analysis in Python**: A Python script is available to analyze marker co-expression across different samples, leveraging custom algorithms to identify patterns and correlations in the data.

### Requirements

- **Steinbock**: Required for image extraction and segmentation. Instructions and more information can be found [here](https://bodenmillergroup.github.io/steinbock).
- **IMCDataAnalysis**: A Docker container required for comprehensive data analysis. More details can be found [here](https://bodenmillergroup.github.io/IMCDataAnalysis/).

### Notes

Ensure you have the necessary dependencies installed and that the environment is properly configured for running both R and Python scripts.

Feel free to reach out for any questions or contributions!
