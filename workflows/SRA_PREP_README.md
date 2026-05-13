# SRA Preparation Workflow (sra_prep)

## Overview

The `sra_prep` workflow prepares standardized metadata tables for submitting sequencing data to the NCBI Sequence Read Archive (SRA). It takes sample information from a Terra workspace table and generates three metadata tables:

1. **SRA Metadata Table** - Formatted for direct NCBI SRA submission
2. **Microbe 1.0 Metadata Table** - NCBI Microbe 1.0 BioSample format
3. **OneHealth Metadata Table** - One Health sample collection metadata format

This workflow is designed to work with the [Terra](https://terra.bio/) platform but can also be run locally with Cromwell.

## Features

- Exports data from Terra workspace tables via the Terra API
- Filters samples by specified sample IDs
- Generates standardized metadata in multiple formats
- Populates default values for sequencing library metadata
- Prepares file paths for bulk transfer of sequencing reads to GCP bucket
- Captures version information and timestamps for reproducibility

## Prerequisites

### For Local Execution
- [Cromwell](https://github.com/broadinstitute/cromwell) (v50+)
- [Docker](https://www.docker.com/) or compatible container runtime
- Python 3.7+
- `pandas` and `numpy` Python libraries
- [Google Cloud SDK](https://cloud.google.com/sdk) (gcloud, gsutil) - for GCP bucket operations

### For Terra Execution
- Access to a Terra workspace
- Read access to the source table
- Write access to the workspace for outputs

### Data Requirements
- A Terra workspace table containing sample information including:
  - Sample ID column
  - Sample name column
  - Organism information column
  - Read file paths (`read1`, `read2` columns)
  - Collection/isolation metadata (dates, locations, sources)

## Inputs

### Required Inputs
| Parameter | Type | Description |
|-----------|------|-------------|
| `terra_project_name` | String | Terra project ID (e.g., "your-project") |
| `workspace_name` | String | Terra workspace name |
| `table_name` | String | Name of the table to export from Terra (e.g., "samples") |
| `sample_names` | Array[String] | List of sample IDs to include in metadata preparation |
| `sra_transfer_gcp_bucket` | String | GCS bucket URI where reads will be copied (e.g., "gs://your-bucket/sra-reads/") |
| `bioproject` | String | NCBI BioProject accession (e.g., "PRJNA123456") |
| `CollectedBy` | String | Name/organization that collected the samples |
| `SequencedBy` | String | Name/organization that performed sequencing |
| `narms_project_name` | String | NARMS project name for metadata |

### Optional Inputs
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `library_strategy` | String | "WGS" | Sequencing strategy (WGS, WXS, RNA-Seq, etc.) |
| `library_source` | String | "GENOMIC" | Source of DNA (GENOMIC, TRANSCRIPTOMIC, etc.) |
| `library_selection` | String | "RANDOM" | Library selection method |
| `library_layout` | String | "paired" | Single or paired-end reads |
| `platform` | String | "ILLUMINA" | Sequencing platform |
| `instrument_model` | String | "MiSeq i100" | Specific instrument model |
| `design_description` | String | "Paired-end 2x150 reads" | Library preparation description |
| `filetype` | String | "fastq" | Format of sequencing files |

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `sra_metadata` | File | SRA-formatted metadata table with timestamps |
| `biosample_metadata` | File | OneHealth-formatted biosample metadata table |
| `sra_prep_version` | String | Workflow version (v0.8) |

### Output File Naming
Files are timestamped with format: `YYYYMMDDTHHMMSS`
- `sra_meta_[timestamp].tsv` - SRA submission metadata
- `onehealth_[timestamp].tsv` - OneHealth biosample metadata
- `microbe_[timestamp].tsv` - Microbe 1.0 biosample metadata (internal)

## Running the Workflow

### Option 1: Local Execution with Cromwell

#### 1. Download Cromwell
```bash
wget https://github.com/broadinstitute/cromwell/releases/download/85/cromwell-85.jar
```

#### 2. Create an inputs JSON file (`inputs.json`)
```json
{
  "sra_prep.terra_project_name": "your-gcp-project",
  "sra_prep.workspace_name": "your-workspace",
  "sra_prep.table_name": "samples",
  "sra_prep.sample_names": ["SAM001", "SAM002", "SAM003"],
  "sra_prep.sra_transfer_gcp_bucket": "gs://your-bucket/sra-reads/",
  "sra_prep.bioproject": "PRJNA123456",
  "sra_prep.CollectedBy": "CDC",
  "sra_prep.SequencedBy": "CDC",
  "sra_prep.narms_project_name": "NARMS_2024",
  "sra_prep.library_strategy": "WGS",
  "sra_prep.library_source": "GENOMIC",
  "sra_prep.platform": "ILLUMINA",
  "sra_prep.instrument_model": "MiSeq"
}
```

#### 3. Run Cromwell
```bash
java -jar cromwell-85.jar run workflows/wf_sra_prep.wdl -i inputs.json
```

Output will be in `cromwell-executions/sra_prep/[execution-id]/`

### Option 2: Terra Execution

1. Create a new method/workflow in Terra using `wf_sra_prep.wdl`
2. Set the workflow input configuration with your parameters
3. Select samples or specify sample IDs in the workflow inputs
4. Launch the workflow
5. Monitor execution in the Jobs tab

### Option 3: WDL IDE (e.g., Dockstore)

The workflow can be run through the [Dockstore](https://dockstore.org) platform using the `.dockstore.yml` configuration file.

## Local Modifications for Non-Terra Use

If running locally without Terra API access, you'll need to:

1. **Replace Terra API call** - The workflow currently uses:
   ```bash
   python3 /scripts/export_large_tsv/export_large_tsv.py
   ```
   Provide your own table export or modify to read from a local TSV file.

2. **Provide input table** - Create a local TSV file with the required columns:
   - `[table_name]_id` - Sample identifier
   - Sample name column
   - Organism column
   - `read1`, `read2` - Paths to sequencing files

3. **GCP access** - For copying files to GCP bucket, ensure `gsutil` is configured with appropriate credentials.

## Input Table Format

### Required Parameters
- `terra_project_name` - Your GCP project ID
- `workspace_name` - Target Terra workspace name
- `table_name` - Source data table name
- `sample_names` - List of sample IDs to process
- `sra_transfer_gcp_bucket` - GCS bucket for read files
- `bioproject` - NCBI BioProject accession
- `CollectedBy` - Organization that collected samples
- `SequencedBy` - Organization that performed sequencing
- `submission_id_column_name` - Sample ID name from the Terra table

### Optional Parameters
- `library_strategy` (default: "WGS")
- `library_source` (default: "GENOMIC")
- `library_selection` (default: "RANDOM")
- `library_layout` (default: "paired")
- `platform` (default: "ILLUMINA")
- `instrument_model` (default: "MiSeq i100")
- `design_description` (default: "Paired-end 2x150 reads")
- food origin, food product type (for foodborne pathogen data)


## Sample Input

Create an `inputs.json` file:
```json
{
  "sra_prep.terra_project_name": "my-gcp-project",
  "sra_prep.workspace_name": "my-workspace",
  "sra_prep.table_name": "samples",
  "sra_prep.sample_names": ["SAM001", "SAM002", "SAM003"],
  "sra_prep.sra_transfer_gcp_bucket": "gs://my-bucket/sra-reads/",
  "sra_prep.bioproject": "PRJNA123456",
  "sra_prep.CollectedBy": "Organization that collected samples",
  "sra_prep.SequencedBy": "Organization that collected samples",
  "sra_prep.narms_project_name": "NARMS_2024",
  "sra_prep.library_strategy": "WGS",
  "sra_prep.platform": "ILLUMINA",
  "sra_prep.instrument_model": "MiSeq"
}
```


## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `sra_metadata` | File | SRA-formatted metadata (timestamped TSV) |
| `biosample_metadata` | File | OneHealth-formatted metadata (timestamped TSV) |
| `sra_prep_version` | String | Workflow version |

Files are timestamped with format: `YYYYMMDDTHHMMSS`

## Usage Examples


### Example: Terra Execution
1. Import `wf_sra_prep.wdl` to Terra workspace using the [link](https://dockstore.org/workflows/github.com/neranjan007/terra_tools/SRA_prep:main?tab=info)
2. Create new workflow in workspace
3. Configure inputs with your sample data
4. Launch through Terra 


## Example Workflow

```bash
# 1. Prepare your input data in Terra or as a local TSV
# 2. Create inputs.json with your parameters
# 3. Run the workflow
java -jar cromwell-85.jar run wf_sra_prep.wdl -i inputs.json

# 4. Find results in cromwell-executions/
ls cromwell-executions/sra_prep/*/call-prep_tables/outputs/

# 5. Review the metadata tables
head sra_meta_20240422T120000.tsv
head onehealth_20240422T120000.tsv
```

## Workflow Structure

```
workflows/
├── wf_sra_prep.wdl          (Main workflow)
└── tasks/
    ├── task_table_prep.wdl   (Metadata preparation logic)
    └── task_version.wdl      (Version/timestamp capture)
```

## Docker Image

The workflow uses a custom Docker image with Terra tools:
- `us-docker.pkg.dev/general-theiagen/theiagen/terra-tools:2023-03-16`
- Contains Python, pandas, numpy, and Terra CLI tools
- Configured for GCP operations via gsutil

## Troubleshooting

### Common Issues

**Issue**: "Cannot find export_large_tsv.py"
- **Solution**: Script location is hardcoded to `/scripts/export_large_tsv/`. For local execution, modify the workflow or provide this script in the Docker image.

**Issue**: Column name errors in pandas operations
- **Solution**: Ensure your table contains the correct column names. Check `submission_id_column_name` and `organism_column_name` parameters in the task.

**Issue**: GCP bucket access denied
- **Solution**: Verify gsutil authentication: `gsutil auth list` and `gsutil config`

**Issue**: Docker image pull fails
- **Solution**: Ensure Docker is running and you have network access. The image may not be available in your environment; use a local alternative.

## Output Usage

### SRA Metadata Table
- Use directly in NCBI SRA submission portal
- Contains library preparation details and file references
- Compatible with NCBI submission templates

### OneHealth Metadata Table
- Formatted for one health surveillance submissions
- Includes detailed collection and isolation metadata
- Suitable for public health databases

## Version History

- **v0.1** - Current version
  - Multiple metadata format support
  - GCP integration for file management
  - Terra workspace integration

## Contributors

CT-DPH Bioinformatics

## License

See LICENSE file in repository root

## References

- [NCBI SRA Submission Guide](https://www.ncbi.nlm.nih.gov/sra/docs/submissionguide/)
- [NCBI Microbe 1.0 BioSample Package](https://www.ncbi.nlm.nih.gov/biosample/docs/packages/)
- [Terra Workspace Documentation](https://support.terra.bio/)
- [Cromwell WDL Execution](https://cromwell.readthedocs.io/)
