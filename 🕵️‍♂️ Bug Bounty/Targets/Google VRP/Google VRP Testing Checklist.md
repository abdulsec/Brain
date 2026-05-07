## Initial Setup (Day 1)
- [ ] Testing Environment Setup
  - [ ] Install/Configure Burp Suite or Caido
  - [ ] Set up browser with testing extensions:
    - [ ] DomLogger++
    - [ ] postMessage-tracker
    - [ ] Wappalyzer
  - [ ] Configure a clean browser profile for testing
  - [ ] Set up Obsidian/Notion for note-taking

- [ ] Google VRP Preparation
  - [ ] Read current VRP rules thoroughly
  - [ ] Review recent Google VRP reports on HackerOne
  - [ ] List target Google products to focus on
  - [ ] Create separate project spaces for each product

## Daily Startup Routine
- [ ] Set specific goals for the day
  - [ ] Write down 3 main targets/features to test
  - [ ] Note which bug classes to focus on
  - [ ] Review previous day's notes

- [ ] Tool Preparation
  - [ ] Start proxy tool (Burp/Caido)
  - [ ] Clear browser cache/cookies
  - [ ] Check all extensions are working
  - [ ] Start new logging session

## Reconnaissance (Days 2-4)
- [ ] Subdomain Enumeration
  - [ ] Run Subfinder
  - [ ] Document all discovered subdomains
  - [ ] Check for wildcard domains
  - [ ] Map relationships between subdomains

- [ ] Asset Discovery
  - [ ] Run httpx on discovered subdomains
  - [ ] Create list of active hosts
  - [ ] Categorize by product/service
  - [ ] Note interesting endpoints

- [ ] URL Discovery
  - [ ] Run Waymore
  - [ ] Run Gau
  - [ ] Filter for interesting parameters
  - [ ] Document API endpoints

## Testing Workflow (Days 5-14)
### Client-Side Testing
- [ ] DOM Analysis
  - [ ] Check for reflected parameters
  - [ ] Test for XSS in reflected data
  - [ ] Look for DOM clobbering opportunities
  - [ ] Test postMessage handlers

- [ ] JavaScript Review
  - [ ] Download all JS files
  - [ ] Look for hidden endpoints
  - [ ] Check for sensitive information
  - [ ] Note interesting functions

### Feature Testing
- [ ] Authentication
  - [ ] Test OAuth flows
  - [ ] Check SSO implementations
  - [ ] Look for token handling issues
  - [ ] Test password reset flows

- [ ] Access Controls
  - [ ] Test IDOR scenarios
  - [ ] Check cross-product permissions
  - [ ] Test resource sharing features
  - [ ] Verify role restrictions

- [ ] Data Handling
  - [ ] Test file upload features
  - [ ] Check data import/export
  - [ ] Look for race conditions
  - [ ] Test cache behaviors

### API Testing
- [ ] Endpoint Analysis
  - [ ] Map all API endpoints
  - [ ] Test parameter handling
  - [ ] Check error responses
  - [ ] Look for logic flaws

- [ ] Integration Testing
  - [ ] Test cross-service interactions
  - [ ] Check data synchronization
  - [ ] Test webhook implementations
  - [ ] Verify callback handling

## Documentation (Ongoing)
### For Each Test
- [ ] Record
  - [ ] Test case details
  - [ ] Steps performed
  - [ ] Tools used
  - [ ] Parameters tested

- [ ] Document Findings
  - [ ] Screenshot evidence
  - [ ] HTTP requests/responses
  - [ ] Reproduction steps
  - [ ] Impact assessment

## End of Day Review
- [ ] Summarize Progress
  - [ ] Document completed tests
  - [ ] Note potential leads
  - [ ] List blocked paths
  - [ ] Plan next day's focus

- [ ] Report Writing (if findings)
  - [ ] Clear title
  - [ ] Detailed description
  - [ ] Reproduction steps
  - [ ] Impact analysis
  - [ ] Suggested fix

## Weekly Review
- [ ] Progress Assessment
  - [ ] Review all notes
  - [ ] Organize findings
  - [ ] Update testing strategies
  - [ ] Identify gaps in coverage

- [ ] Planning
  - [ ] Adjust focus areas
  - [ ] Update target list
  - [ ] Refine methodology
  - [ ] Set next week's goals

## Safety Checks
- [ ] Before Every Test
  - [ ] Verify in-scope target
  - [ ] Check for rate limiting
  - [ ] Avoid DoS scenarios
  - [ ] Monitor resource usage

- [ ] After Finding a Bug
  - [ ] Stop testing affected component
  - [ ] Document everything immediately
  - [ ] Verify reproducibility
  - [ ] Check for duplicate reports

## Report Quality Checks
- [ ] Before Submission
  - [ ] Clear description
  - [ ] Step-by-step reproduction
  - [ ] Clean screenshots/videos
  - [ ] Impact clearly stated
  - [ ] Technical details accurate
  - [ ] Proofread everything