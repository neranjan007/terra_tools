# terra_tools

A collection of WDL workflows and reusable tasks for bioinformatics data processing and metadata preparation on the [Terra](https://terra.bio/) platform. These workflows are designed for genomic sequencing data processing and NCBI submission preparation.

## Overview

The `terra_tools` project provides production-ready WDL workflows for handling genomic data pipelines, with a focus on:

- **NCBI SRA Metadata Preparation** - Standardized formatting for Sequence Read Archive submissions
- **Modular Task Design** - Reusable WDL task components for common bioinformatics operations
- **Terra Platform Integration** - Built for seamless execution on Terra with optional local Cromwell support
- **Reproducibility** - Version tracking and timestamped outputs for audit trails

## Quick Start

### For Terra Users

1. **Import the Workflow**
   - Go to [Dockstore](https://dockstore.org) and search for this repository
   - Select "Import" to add workflows to your Terra workspace


2. **Configure Inputs**
   - Set required parameters (see [Inputs](#inputs) section)
   - Select samples from your workspace table
   - Configure sequencing metadata (library strategy, platform, etc.)

3. **Launch Workflow**
   - Click "Run" in the Terra Jobs tab
   - Monitor execution progress
   - Download results when complete



## Project Structure

```
terra_tools/
├── README.md                          # This file
├── LICENSE                            # GNU General Public License v3
├── .dockstore.yml                     # Dockstore configuration
├── empty.json                         # Empty test parameters
│
├── workflows/
│   ├── wf_sra_prep.wdl               # SRA metadata preparation workflow
│   └── SRA_PREP_README.md            # Detailed SRA workflow documentation
│
└── tasks/
    ├── task_table_prep.wdl           # Core table preparation task
    └── task_version.wdl              # Version capture task
```

### Workflow Details

#### `wf_sra_prep.wdl` - SRA Preparation Workflow
Prepares standardized metadata tables for NCBI SRA submission. Takes sample information from a Terra workspace and generates three formatted metadata tables:
- SRA submission format
- NCBI Microbe 1.0 BioSample format  
- OneHealth sample collection format

See [SRA_PREP_README.md](workflows/SRA_PREP_README.md) for comprehensive documentation.

### Task Details

#### `task_table_prep.wdl`
Core task for processing and formatting genomic sample metadata into standardized formats.

#### `task_version.wdl`
Captures version information and timestamps for workflow reproducibility.

## Dependencies & Requirements

### For Terra Execution
- Active Terra account with workspace access
- Read access to source data tables
- Write permissions to output workspace

### Data Requirements
- Terra workspace table containing:
  - Sample identifiers
  - Sample metadata (organism, collection date/location, source)
  - Read file paths (`read1`, `read2` columns for paired-end data)

## Inputs

For detailed input specifications, see [SRA_PREP_README.md](workflows/SRA_PREP_README.md#inputs)

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

### Example 1: Local Execution with Cromwell
```bash
cd /path/to/terra_tools
java -jar cromwell-85.jar run workflows/wf_sra_prep.wdl -i my_inputs.json
```

### Example 2: Terra Execution
1. Upload `wf_sra_prep.wdl` to Terra Method Repository
2. Create new workflow in workspace
3. Configure inputs with your sample data
4. Click "Run workflow"

### Example 3: Dockstore
1. Visit https://dockstore.org
2. Search for "terra_tools"
3. Launch through Terra or local Cromwell

## Contributing Guidelines

We welcome contributions! Please follow these guidelines:

### Code Style
- Follow WDL best practices from [OpenWDL.org](https://www.openwdl.org/)
- Use meaningful variable and task names
- Add comments for complex logic
- Keep tasks modular and reusable

### Testing
- Test workflows locally with Cromwell before submission
- Use test inputs (`empty.json` as template)
- Verify outputs match expected formats
- Test with both small and large sample sets

### Documentation
- Update relevant README files
- Document new inputs/outputs
- Provide usage examples
- Include version numbers in commit messages

### Submitting Changes
1. Create a feature branch
2. Make your changes
3. Test thoroughly
4. Submit a pull request with clear description
5. Ensure CI/CD checks pass

## Support & Contact

- **Questions?** Check the detailed documentation in [workflows/SRA_PREP_README.md](workflows/SRA_PREP_README.md)
- **Issues?** Report bugs and feature requests on this repository
- **Documentation**: Review WDL documentation at https://www.openwdl.org/
- **Terra Support**: Visit https://support.terra.bio/

## License

This project is licensed under the **GNU General Public License v3.0** - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built with [WDL](https://www.openwdl.org/) (Workflow Description Language)
- Designed for [Terra](https://terra.bio/) platform
- Supports [NCBI Sequence Read Archive](https://www.ncbi.nlm.nih.gov/sra/) submissions

---

**Last Updated**: April 2026
**Current Version**: v0.8