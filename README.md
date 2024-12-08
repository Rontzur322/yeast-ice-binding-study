# yeast-ice-binding-study

This repository provides video analysis workflows and data visualization scripts for tracking yeast pathways during directional freezing experiments. By integrating **Fiji (ImageJ2)** for video preprocessing, **Python** for data preparation, and **RStudio** for statistical analysis, this systematic pipeline enables quantitative analysis of ice-binding protein effects on yeast engulfment in ice.

---

## Features
- **Fiji Preprocessing**: Standardized video import, filtering, and calibration for trajectory analysis.
- **TrackMate Tracking**: Accurate detection and tracking of yeast cells using the Laplacian of Gaussian (LoG) Detector.
- **Data Preparation**: Python scripts consolidate and extract the Max Distance Traveled parameter.
- **Statistical Analysis**: Visualization and statistical tests in RStudio to assess yeast displacement behavior.

---

## Repository Structure

yeast-ice-binding-study/

protocols/                 # Experimental protocols

data/                      # Raw and consolidated datasets

data_for_violin.csv

scripts/                   # Scripts for analysis and visualization

Fiji_Macro.ijm         # Fiji macro for preprocessing videos

process_data.py        # Python script for data consolidation

violin_plot.R          # R script for violin plots and statistical analysis

results/                   # Outputs from analysis

violin_plot.png        # Violin plot for yeast displacement

statistics_output.txt  # Statistical analysis results

README.md                  # Project overview and workflow instructions

---

## Workflow

### **2.3.1 Fiji: Video Preprocessing and Tracking**
1. **Import Videos**:
   - Use the **AVI Reader** plugin in Fiji with the following settings:
     - Use Virtual Stack: **Checked**.
     - Convert to Grayscale: **Checked**.

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
     - Standardizes grayscale conversion.
     - Adds spatial calibration (420 microns = 1024 pixels).
     - Noise reduction with variance and median filters.

3. **Yeast Detection and Tracking in TrackMate**:
   - **Detector**:
     - Select: **Laplacian of Gaussian (LoG)**.
     - Estimated Object Radius: **7 µm**.
     - Quality Threshold: **3** (refined to **5.01**).
   - **Tracking**:
     - Tracker: **LAP Tracker**.
     - Frame-to-Frame Linking: Max Distance = **5 µm**.
     - Gap Closing: Max Distance = **5 µm**, Max Frame Gap = **3**.
   - **Displacement Filtering**:
     - Apply a minimum displacement filter: **200 µm**.

4. **Output**:
   - Export the filtered yeast tracks to a CSV file:
     - File → Export → **Export Tracks to CSV**.

---

### **2.3.2 Python: Data Preparation**
The tracking CSV files are consolidated into a single dataset using Python.

#### Script: `scripts/process_data.py`
```python
import pandas as pd
import os

# Folder path for raw tracking CSVs
input_folder = "data/raw/"
output_file = "data/data_for_violin.csv"

# Consolidate all tracking data
dataframes = []
for file in os.listdir(input_folder):
    if file.endswith(".csv"):
        df = pd.read_csv(os.path.join(input_folder, file))
        max_distance = df['MAX_DISTANCE_TRAVELED']
        group = file.split("_")[0]  # Assumes file names include protein group
        consolidated = pd.DataFrame({'Group': group, 'Value': max_distance})
        dataframes.append(consolidated)

# Combine all datasets
final_df = pd.concat(dataframes, ignore_index=True)
final_df = final_df[final_df['Value'] > 0]  # Remove invalid values

# Export to CSV
final_df.to_csv(output_file, index=False)
print("Data preparation complete. Output saved to:", output_file)

Purpose:
	•	Combines individual CSV files into one clean dataset.
	•	Extracts the Max Distance Traveled parameter.
	•	Classifies yeast by protein group and removes immobile cells.

2.3.3 RStudio: Statistical Analysis and Visualization

The prepared dataset is imported into RStudio for statistical analysis and visualization.

Script: scripts/violin_plot.R

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

Statistical Tests:
	•	Normality: Shapiro-Wilk test.
	•	Homogeneity: Levene’s test.
	•	Parametric data: One-way ANOVA with Tukey’s post-hoc test.
	•	Non-parametric data: Kruskal-Wallis test with Dunn’s test.

Outputs
	•	Tracking Results: Exported CSV files of yeast pathways.
	•	Consolidated Data: Clean dataset for visualization (data_for_violin.csv).
	•	Violin Plot: A plot summarizing yeast displacement (results/violin_plot.png).

Requirements
	•	Fiji (ImageJ2): Version 2.14.0.
	•	Python: With Pandas library.
	•	RStudio: With ggplot2 package.

License

This repository is licensed under the MIT License.

Contact

Ron Tzur
[Insert Email Here]

Acknowledgments

Special thanks to the developers of Fiji (ImageJ2), TrackMate, Python, and RStudio for enabling rigorous and reproducible analysis.

---

### Steps to Use This:
1. Save this content as `README.md` in the root of your repository.
2. Commit and push it:
   ```bash
   git add README.md
   git commit -m "Add finalized README with full pipeline details"
   git push origin main
