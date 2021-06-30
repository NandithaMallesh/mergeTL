# Merge TL: Transfer learning on merged FCS data.
Workflow to extend AI model trained on a specific MFC panel to multiple data sets.

### Worflow

![Workflow](https://user-images.githubusercontent.com/22116058/123971644-85fc9700-d9ba-11eb-9b88-910c88e45982.png)

Step 1: Merge FCS files for each of the data set

Step 2: Generate SOMs

Step 3: Train CNN models for each data set

### overview
* Train a base model using the base data set. 
* For each target data set, 2 models are generated: 
	1) standalone model (no transfer learning)
	2) model with transfer learning (Initilaized with weights from base model)

### System Requirements
The SOM generation and classification both require tensorflow, a recent NVIDIA GPU is preferable. The system has been internally used using a P40 GPU.

Packages needed:

* flowcat
* fcsmerge

Other dependencies: Tensforflow=1.12, python=3.6. All dependencies can be installed using
```
$ pip install /path/to/wheel
```
We suggest using a specific python environment to avoid collisions with other versions of the same python libraries.

### Set up
1) Download all data sets: https://doi.org/10.7910/DVN/CQHHEH
2) clone https://github.com/NandithaMallesh/fcsmerge.git and merge FCS files for each of the data sets
3) clone and install flowcat from: https://github.com/xiamaz/flowCat.git
4) clone this repository

### Data sets used
* Base data set: 9-color panel (MLL9F)
* Target data sets:
	1) MLL5F panel
	2) Berlin panel
	3) Bonn panel
	4) Erlangen panel

### Scripts
1) **mergeTL:**
	1) **generate_ref_som.py:** generate reference SOM for merged base dataset(MLL9F). We will use this reference SOM for generating individual SOMs for each  dataset.
	2) **merged_model.py:** train standalone model for the merged data
	3) **merge_TL.py:** train target models with knowledge transfer from the given base model. create target model, set weights from the base model.
2) **kold:** scripts to generate k-fold results for each data set
3) **figures:** scripts to generate all the figures included in the publication
4) **experiments:** sripts for learning curve experiments
	
### Usage
1) Generate reference SOM for the merged base data. Ensure the input and output paths match your set up in "generate_ref_som.py".
```
 	./generate_ref_som.py
```	
2) Generate SOMs for each data set(both base and target data sets) using flowcat
```
	flowcat transform --data "$DATA" --meta "$META" --reference "$REF_OUTPUT" --output "$SOM_OUTPUT"
	
   e.g.:flowcat transform --data "/data/base/MLL9F" --meta "/data/base/MLL9F/meta.json" --reference "/data/SOM/reference" --output "/data/SOM/MLL9F/"
	
	$DATA: path to the merged FCS files
	$META: path to the meta info json
	$REF_OUTPUT: path to reference SOM (output of the previous command)
	$SOM_OUTPUT: path to save the generated SOMs
	
```
	
3) Train base model
```
   ./merged_model.py "$BASE_SOM_PATH" "$OUTPUT" "$PANEL"
	
   ./merged_model.py "/data/SOM/MLL9F/" "/data/models/base/MLL9F" "MLL"
	
	$BASE_SOM_PATH : path to merged SOM files of the base data set
	$OUTPUT: Out put folder path to save model and meta information
	$PANEL: MLL

 ```
	
4) Train standalone models for each target data set
```
	./merged_model.py "$TARGET_SOM_PATH" "$OUTPUT" "$PANEL" 
	
  e.g.: ./merged_model.py  "/data/SOM/MLL5F/" "/data/models/target/standalone/MLL5F" "MLL"
	
	$TARGET_SOM_PATH : path to merged SOM files of each target data set
	$OUTPUT: Out put folder path to save model and meta information
	$BASEMODEL_PATH: path where the base model is saved (parent folder containing the model.h5 file)
	$PANEL: one of the following
		* MLL, Berlin, Bonn, Erlangen
```
	
5) Train models with transfer learning for each target data set
```
	./merge_TL.py "$TARGET_SOM_PATH" "$OUTPUT" "$PANEL" "$BASEMODEL_PATH"
	
  e.g.: ./merge_TL.py "/data/SOM/MLL5F/" "/data/models/target/standalone/MLL5F" "MLL" "/data/models/base/MLL9F"
	
	$TARGET_SOM_PATH : path to merged SOM files of each target data set
	$OUTPUT: Out put folder path to save model and meta information
	$BASEMODEL_PATH: path where the base model is saved (parent folder containing the model.h5 file)
	$PANEL: one of the following
		* MLL, Berlin, Bonn, Erlangen
```

