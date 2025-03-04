# yeast-ice-binding-study
This repository provides video analysis workflows and data visualization scripts for tracking yeast pathways during directional freezing experiments.
# yeast-ice-binding-study

This repository provides video analysis workflows and data visualization scripts for tracking yeast pathways during directional freezing experiments. By integrating **Fiji (ImageJ2)** for video preprocessing, **Python** for data preparation, and **RStudio** for statistical analysis, this systematic pipeline enables quantitative analysis of ice-binding protein effects on yeast engulfment in ice.

---

## Features
- **Fiji Preprocessing**: Standardized video import, filtering, and calibration for trajectory analysis.
- **TrackMate Tracking**: Accurate detection and tracking of yeast cells using the Laplacian of Gaussian (LoG) Detector.
- **Data Preparation**: Python scripts consolidate and extract the "MAX_DISTANCE_TRAVELED" parameter, reformatting the data into a long format ready for visualization.
- **Statistical Analysis**: Visualization and statistical tests in RStudio to assess yeast displacement behavior.

---

## Repository Structure
---

## Workflow

### 1. Fiji: Video Preprocessing and Tracking

1. **Import Videos**:
   - Use the **AVI Reader** plugin in Fiji with the following settings:
     - **Virtual Stack**: Checked.
     - **Convert to Grayscale**: Checked.

2. **Preprocessing**:
   - Run the custom macro `scripts/Fiji_Macro.ijm`:
     ```java
     setOption("ScaleConversions", true);
     run("8-bit");
     run("Duplicate...", "duplicate");
     run("Set Scale...", "distance=1024 known=420 unit=micron");
     run("Scale Bar...", "width=10 height=20 thickness=5 font=15 bold overlay");
     run("Variance...", "radius=3 stack");
     run("Median...", "radius=3 stack");
     ```
   - **Purpose**:
     - Convert videos to 8-bit grayscale.
     - Duplicate and calibrate images (420 microns = 1024 pixels).
     - Reduce noise via variance and median filtering.

3. **Yeast Detection and Tracking in TrackMate**:
   - **Detector**:
     - **Type**: Laplacian of Gaussian (LoG).
     - **Estimated Object Radius**: 7 µm.
     - **Quality Threshold**: Adjusted to 5.01.
   - **Tracking**:
     - **Tracker**: LAP Tracker.
     - **Frame-to-Frame Linking**: Maximum Distance = 5 µm.
     - **Gap Closing**: Maximum Distance = 5 µm, Maximum Frame Gap = 3.
   - **Displacement Filtering**:
     - Apply a minimum displacement filter of 200 µm.

4. **Output**:
   - Export the filtered yeast tracks to a CSV file:
     - **File → Export → Export Tracks to CSV**.

---

### 2. Python: Data Preparation

The tracking CSV files are consolidated into a single dataset using Python. The final dataset is structured in a long format, where:

- **Column A ("Protein")** contains the protein name (extracted from the filename).
- **Column B ("Value")** contains individual numerical measurements (extracted from the "MAX_DISTANCE_TRAVELED" column).

#### Script: `scripts/process_data.py`
```python
import os
import re
import pandas as pd

# Define paths
source_folder = '/content/drive/MyDrive/DF_DATA'
final_csv_path = '/content/drive/MyDrive/DF_DATA/DF_DATA_FINAL/DF_DATA_FINAL.csv'

# Clear the final CSV file and write header (Protein in column A, Value in column B)
with open(final_csv_path, 'w') as f:
    f.write("Protein,Value\n")

# Process each file in the source folder
for filename in os.listdir(source_folder):
    file_path = os.path.join(source_folder, filename)
    
    # Skip directories (e.g., DF_DATA_FINAL)
    if not os.path.isfile(file_path):
        continue

    print(f"Processing file: {filename}")
    
    # Extract the protein name from the filename (everything before the first underscore)
    protein_name = filename.split('_')[0]
    
    try:
        # Read the CSV file (assuming headers are in the first row)
        df = pd.read_csv(file_path)
    except Exception as e:
        print(f"Error reading {filename}: {e}")
        continue

    # Check if the target column "MAX_DISTANCE_TRAVELED" exists
    if 'MAX_DISTANCE_TRAVELED' not in df.columns:
        print(f"Column 'MAX_DISTANCE_TRAVELED' not found in {filename}. Skipping file.")
        continue

    # Extract numbers from each cell in the "MAX_DISTANCE_TRAVELED" column
    numbers_extracted = []
    for cell in df['MAX_DISTANCE_TRAVELED']:
        if pd.isnull(cell):
            continue
        cell_str = str(cell)
        # Use regex to extract numbers (including decimals)
        matches = re.findall(r"[-+]?\d*\.\d+|[-+]?\d+", cell_str)
        numbers_extracted.extend(matches)
    
    # Append each number as a separate row in the final CSV:
    # Column A is the protein name, Column B is one extracted value.
    with open(final_csv_path, 'a') as f:
        for number in numbers_extracted:
            f.write(f"{protein_name},{number}\n")
    
    print(f"Finished processing file: {filename}")

print("All files processed and final CSV updated.")
# Load ggplot2 library
if (!requireNamespace("ggplot2", quietly = TRUE)) {
  install.packages("ggplot2")
}
library(ggplot2)

# Load dataset
data <- read.csv('data/data_for_violin.csv')

# Generate violin plot
plot <- ggplot(data, aes(x = Group, y = Value, fill = Group)) +
  geom_violin(trim = FALSE, alpha = 0.5) +
  geom_jitter(aes(color = Group), width = 0.2, size = 1, alpha = 0.7) +
  geom_hline(yintercept = 350, linetype = "dashed", color = "black") +
  annotate("text", x = 0.5, y = 365, label = "Engulfment Threshold", size = 4) +
  theme_classic() +
  labs(
    title = "Maximum Distance Traveled by Yeast Cells",
    x = "Protein Group",
    y = "Distance (µm)"
  )

# Save plot
ggsave("results/violin_plot.png", plot, width = 10, height = 6, dpi = 300)
git clone https://github.com/yourusername/yeast-ice-binding-study.git
cd yeast-ice-binding-study
git add README.md
git commit -m "Update README with full pipeline details"
git push origin main
Simply copy and paste the above content into your `README.md` file in your GitHub repository.
