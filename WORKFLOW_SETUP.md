# AI Issue Analysis Workflow Setup

This guide will help you set up the automated AI issue analysis workflow for your repository.

## Prerequisites

1. **Repository with Issues enabled**: This workflow is designed to run when new issues are created
2. **Gemini AI API Key**: You'll need a Google Gemini API key for the AI analysis
3. **GitHub Actions enabled**: The repository must have GitHub Actions enabled

## Setup Steps

### 1. Add the Workflow File

The workflow file `.github/workflows/ai-issue-analysis.yml` has been created in your repository. This file contains all the automation logic.

### 2. Configure the Gemini API Key

1. **Get a Gemini API Key**:
   - Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
   - Create a new API key
   - Copy the API key

2. **Add the API Key to GitHub Secrets**:
   - Go to your repository on GitHub
   - Navigate to `Settings` → `Secrets and variables` → `Actions`
   - Click `New repository secret`
   - Name: `GEMINI_API_KEY`
   - Value: Your Gemini API key
   - Click `Add secret`

### 3. Verify Permissions

The workflow requires the following permissions (these are already configured in the workflow file):
- `issues: write` - To post comments on issues
- `contents: read` - To read repository contents

### 4. Test the Workflow

1. **Create a test issue** in your repository
2. **Monitor the workflow**:
   - Go to the `Actions` tab in your repository
   - Look for a workflow run named "AI Issue Analysis"
   - Click on it to see the progress and logs

## How It Works

### Workflow Trigger
- Triggers automatically when a new issue is created (`issues: opened`)

### Workflow Steps
1. **Environment Setup**: Sets up Node.js 18 and Python 3.11
2. **Install repomix**: Installs repomix globally via npm
3. **Generate repomix output**: Runs repomix remotely on ansible-creator repository to create a text representation
4. **Clone AI-Issue-Triage**: Gets the AI analysis tool
5. **Install dependencies**: Installs Python packages for the AI tool
6. **Run analysis**: Executes the AI analysis using the issue title, description, and codebase
7. **Post comment**: Adds the analysis results as a comment on the original issue
8. **Upload artifacts**: Saves analysis files for later review

### Expected Output

When an issue is created, you should see:
- A new comment on the issue with AI analysis results
- Analysis artifacts uploaded to the workflow run
- The analysis includes:
  - Issue classification (bug, enhancement, feature request)
  - Severity assessment
  - Root cause analysis
  - Proposed solutions with code suggestions

## Customization Options

### Change Target Repository
To analyze a different repository instead of ansible-creator, modify this section in the workflow:

```yaml
- name: Generate repomix output for ansible-creator
  run: |
    echo "Running repomix on repository using remote..."
    repomix --remote https://github.com/ansible/ansible-creator --output repomix-output.txt  # Change this URL
    echo "Repomix output generated successfully"
    ls -la repomix-output.txt
```

### Modify Analysis Parameters
You can adjust the CLI parameters in the "Run AI issue analysis" step:

```yaml
python cli.py \
  --title "${{ steps.issue-content.outputs.title }}" \
  --description "${{ github.event.issue.body }}" \
  --format json \
  --output analysis_result.json \
  --quiet
```

### Custom Comment Format
Modify the comment format in the "Post analysis as issue comment" step to change how results are displayed.

## Troubleshooting

### Common Issues

1. **Missing API Key**:
   - Error: "GEMINI_API_KEY environment variable not set"
   - Solution: Ensure the secret is properly configured in repository settings

2. **Workflow Permission Denied**:
   - Error: "Resource not accessible by integration"
   - Solution: Check that the workflow has proper permissions in the YAML file

3. **Repomix Installation Fails**:
   - Error: "npm install -g repomix failed"
   - Solution: This is usually a temporary npm issue; re-run the workflow

4. **Analysis Timeout**:
   - Error: Analysis takes too long or times out
   - Solution: Consider using a smaller repository or adjusting the repomix output

### Viewing Logs

1. Go to the `Actions` tab in your repository
2. Click on the failed workflow run
3. Expand the failing step to see detailed error messages
4. Check the "Upload analysis artifacts" step for saved files

### Testing Locally

You can test the analysis locally before relying on the workflow:

```bash
# Install repomix
npm install -g repomix

# Generate repomix output using remote repository
repomix --remote https://github.com/ansible/ansible-creator --output repomix-output.txt

# Clone the AI tool
git clone https://github.com/shvenkat-rh/AI-Issue-Triage
cd AI-Issue-Triage
pip install -r requirements.txt

# Run analysis
export GEMINI_API_KEY="your-api-key"
python cli.py --title "Test Issue" --description "Test description" --source-path ../repomix-output.txt
```

## Cost Considerations

- **GitHub Actions**: Free tier includes 2,000 minutes/month for public repositories
- **Gemini API**: Costs vary based on usage; check [Google's pricing](https://ai.google.dev/pricing)
- **Storage**: Artifacts are stored for 30 days and count toward your storage quota

## Security Notes

- The Gemini API key is stored as a GitHub secret and not exposed in logs
- The workflow only has read access to repository contents
- All analysis is performed in isolated GitHub Actions runners

## Support

If you encounter issues:
1. Check the workflow logs in the Actions tab
2. Verify your API key is correctly configured
3. Ensure the target repository (ansible-creator) is accessible
4. Review the AI-Issue-Triage repository for any updates or known issues
