# PET to BIDS

## History

The PET modality is one of the newest additions to the BIDS standard with its introduction via BEP 009. If you're
interested in seeing exactly what and how something gets added to BIDS see the pull request for
[BEP 009 here](https://github.com/bids-standard/bids-specification/pull/633). The results of that extension proposal can
be
read [here](https://bids-specification.readthedocs.io/en/stable/04-modality-specific-files/09-positron-emission-tomography.html#positron-emission-tomography)
in the bids standard.

## Conversion

Presently, PET does not have the same level of support when it comes to conversion from raw
data into BIDS format like MR imaging. Much of the work in converting from a raw PET dataset
into a bids PET dataset will be incumbent on the user. Some useful tools and resources 
that will be used in this document are as follows: 

- [BIDS Validator](https://github.com/bids-standard/bids-validator), which fully supports PET

- [TPCCLIIB](https://gitlab.utu.fi/vesoik/tpcclib) is a command line library containing (among many others) PET tools
such as `ecat2nii` that will be used below to convert the imaging data from a PET dataset into nifti format. 
  The Turku PET Centre site can be found [here](https://turkupetcentre.fi/) for additional information on anything PET.
    
- a [BIDS PET Template](https://github.com/bids-standard/bids-starter-kit/tree/master/templates)
from this starter kit to initially populate and translate text/tabulature/csv blood data into
  BIDS PET compliant `.tsv` and `.json` files.
  
### Setting the layout

This starter kit provides several template/example files that can be a great starting place to help
get set up for the conversion process. PET specific text data files can be viewed at 
[this location](https://github.com/bids-standard/bids-starter-kit/tree/master/templates/sub-01/ses-01/pet).
And easily collected via this [link](https://github.com/bids-standard/bids-starter-kit/archive/refs/heads/master.zip)
or be cloned and extracted via git at the command line via:

```bash
git clone git@github.com:bids-standard/bids-starter-kit.git
```

The `templates/` folder in `bids-starter-kit/` contains a single subject and examples of every BIDS modality
text and `.json` file for that subject. Additionally, there exists a `Short` and a `Full` example for each
text/json file. The `Short` files contain all of the required BIDS fields for each modality and the `Full` 
example files contain *every* field included within the BIDS standard.

```bash
templates/
├── README
├── dataset_description.json
├── participants.json
├── participants.tsv
├── phenotype
│   ├── EpilepsyClassification.tsv
│   └── EpilepsyClassification2017.json
└── sub-01
    └── ses-01
        ├── anat
        │   ├── sub-01_ses-01_acq-FullExample_run-01_T1w.json
        │   └── sub-01_ses-01_acq-ShortExample_run-01_T1w.json
        ├── eeg
        │   ├── sub-01_ses-01_task-FilterExample_eeg.json
        │   ├── sub-01_ses-01_task-FullExample_eeg.json
        │   ├── sub-01_ses-01_task-MinimalExample_eeg.json
        │   └── sub-01_ses-01_task-ReferenceExample_eeg.json
        ├── fmap
        │   ├── sub-01_ses-01_task-Case1_run-01_phasediff.json
        │   ├── sub-01_ses-01_task-Case2_run-01_phase1.json
        │   ├── sub-01_ses-01_task-Case2_run-01_phase2.json
        │   ├── sub-01_ses-01_task-Case3_run-01_fieldmap.json
        │   └── sub-01_ses-01_task-Case4_dir-LR_run-01_epi.json
        ├── func
        │   ├── sub-01_ses-01_task-FullExample_run-01_bold.json
        │   ├── sub-01_ses-01_task-FullExample_run-01_events.json
        │   ├── sub-01_ses-01_task-FullExample_run-01_events.tsv
        │   └── sub-01_ses-01_task-ShortExample_run-01_bold.json
        ├── ieeg
        │   ├── sub-01_ses-01_coordsystem.json
        │   ├── sub-01_ses-01_electrodes.tsv
        │   ├── sub-01_ses-01_task-LongExample_run-01_channels.tsv
        │   └── sub-01_ses-01_task-LongExample_run-01_ieeg.json
        ├── meg
        │   ├── sub-01_task-FullExample_acq-CTF_run-1_proc-sss_meg.json
        │   └── sub-01_task-ShortExample_acq-CTF_run-1_proc-sss_meg.json
        └── pet
            ├── sub-01_ses-01_recording-AutosamplerShortExample_blood.json
            ├── sub-01_ses-01_recording-AutosamplerShortExample_blood.tsv
            ├── sub-01_ses-01_recording-ManualFullExample_blood.json
            ├── sub-01_ses-01_recording-ManualFullExample_blood.tsv
            ├── sub-01_ses-01_recording-ManualShortExample_blood.json
            ├── sub-01_ses-01_recording-ManualShortExample_blood.tsv
            ├── sub-01_ses-01_task-FullExample_pet.json
            └── sub-01_ses-01_task-ShortExample_pet.json

10 directories, 35 files
machine:bids-starter-kit user$
```

For the purposes of this exercise we will only be focusing on obtaining the minimum required fields, thus
we collect and copy the following to our own BIDS folder.

```bash
# create the new bids folder
mkdir -p /NewBidsDataSet/sub-01/ses-01/pet
cp -r /path/to/bids-starter-kit/templates/sub-01/ses-01/pet/* /NewBidsDataSet/sub-01/pet/
```

Oops, we forgot to include our dataset specific files. Let's make it easy on ourselves and include the 
templates for those too.

```bash
cp /path/to/bids-starter-kit/templates/* /NewBidsDataSet/
```

Our skeleton of a data set should now look like this:
```bash
machine: Projects user$ tree NewBidsDataSet/
NewBidsDataSet/
├── README
├── dataset_description.json
├── participants.json
├── participants.tsv
└── sub-01
    └── ses-01
        └── pet
            ├── sub-01_ses-01_recording-AutosamplerShortExample_blood.json
            ├── sub-01_ses-01_recording-AutosamplerShortExample_blood.tsv
            ├── sub-01_ses-01_recording-ManualFullExample_blood.json
            ├── sub-01_ses-01_recording-ManualFullExample_blood.tsv
            ├── sub-01_ses-01_recording-ManualShortExample_blood.json
            ├── sub-01_ses-01_recording-ManualShortExample_blood.tsv
            ├── sub-01_ses-01_task-FullExample_pet.json
            └── sub-01_ses-01_task-ShortExample_pet.json

3 directories, 12 files
machine:Projects user$
```

Now let's do the bare minimum and focus only on the short files:

```bash
machine:Projects user$ rm -rf NewBidsDataSet/sub-01/ses-01/pet/*Full*
machine:Projects user$ tree NewBidsDataSet
NewBidsDataSet/
├── README
├── dataset_description.json
├── participants.json
├── participants.tsv
└── sub-01
    └── ses-01
        └── pet
            ├── sub-01_ses-01_recording-AutosamplerShortExample_blood.json
            ├── sub-01_ses-01_recording-AutosamplerShortExample_blood.tsv
            ├── sub-01_ses-01_recording-ManualShortExample_blood.json
            ├── sub-01_ses-01_recording-ManualShortExample_blood.tsv
            └── sub-01_ses-01_task-ShortExample_pet.json

3 directories, 9 files
machine:Projects user$
```

That certainly looks less daunting, now let's change the filenames of the templates so that they make more
sense for our data set (aka remove ShortExample from each filename). **Note:** if you have multiple PET image files you 
can distinguish between them with the `task-<your description>` label. If there's a single pet scan you may
omit the `task` label from the filename.

```bash
# for those of you in bash land
machine:Projects user$ cd NewBidsDataSet/sub-01/ses-01/pet 
machine:pet user$ for i in *ShortExample*; do mv "$i" "`echo $i | sed 's/ShortExample//'`"; done
# Since we "don't" know if there are multiple PET image files we will omit the task label for our PET .json
# file for now.
machine:pet user$ mv sub-01_ses-01_task-_pet.json sub-01-ses-01_pet.json
```

After you've renamed your files your directory should look something like:

```bash
machine:Projects user$ tree NewBidsDataSet/
NewBidsDataSet/
├── README
├── dataset_description.json
├── participants.json
├── participants.tsv
└── sub-01
    └── ses-01
        └── pet
            ├── sub-01_ses-01_pet.json
            ├── sub-01_ses-01_recording-Autosampler_blood.json
            ├── sub-01_ses-01_recording-Autosampler_blood.tsv
            ├── sub-01_ses-01_recording-Manual_blood.json
            └── sub-01_ses-01_recording-Manual_blood.tsv

3 directories, 9 files
machine:Projects user$
```


### Collecting and installing TPCCLIIB
&& git clone
&& set path etc etc

### Installing the validator
&& just link to methods

### Gathering raw pet data
&&show side by side w/ ideal bids layout
&&

## MISC
&&add link to Remi's guide/walkthrough for collecting PET data.



