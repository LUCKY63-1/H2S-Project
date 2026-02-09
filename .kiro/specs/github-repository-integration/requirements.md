# GitHub Repository Integration - Requirements

## Feature Overview

Enable users to analyze code directly from GitHub repositories by providing a repository URL, eliminating the need for manual file uploads. This feature will clone or fetch repository contents, filter supported code files, and integrate seamlessly with the existing file upload workflow.

---

## User Stories

### 1. As a developer, I want to paste a GitHub repository URL so that I can analyze code without manually downloading and uploading files

**Acceptance Criteria:**
- User can input a GitHub repository URL in the dashboard
- System validates the GitHub URL format
- System fetches/clones the repository contents
- System extracts only supported code file types
- System displays fetched files for user confirmation
- User can proceed with analysis using fetched files

### 2. As a developer, I want to choose between manual upload and GitHub URL so that I have flexibility in how I provide code

**Acceptance Criteria:**
- Dashboard provides clear UI toggle between "Upload Files" and "GitHub Repository" modes
- Only one mode is active at a time
- Switching modes clears the previous selection
- Both modes lead to the same validation and analysis workflow

### 3. As a developer, I want to see which files were fetched from the repository so that I can verify the correct files will be analyzed

**Acceptance Criteria:**
- System displays list of fetched files with metadata (name, size, type, language)
- User can see file count and total size
- User can review files before starting analysis
- System shows validation results for fetched files

### 4. As a developer, I want the system to handle errors gracefully when fetching from GitHub so that I understand what went wrong

**Acceptance Criteria:**
- System validates GitHub URL format before attempting fetch
- System provides clear error messages for:
  - Invalid URLs
  - Repository not found (404)
  - Network errors
  - Authentication errors (private repos)
  - Rate limiting
  - Large repositories
- System suggests corrective actions for each error type

### 5. As a developer, I want to analyze public GitHub repositories without authentication so that I can quickly analyze open-source code

**Acceptance Criteria:**
- System can fetch public repositories without requiring GitHub token
- System uses GitHub API for efficient fetching
- System respects GitHub API rate limits
- System provides feedback on rate limit status

### 6. As a developer, I want the option to provide a GitHub token for private repositories so that I can analyze my private code

**Acceptance Criteria:**
- Dashboard provides optional GitHub token input field
- Token is stored securely in session state
- System uses token for authenticated API requests
- System provides clear feedback when authentication is required
- Token is not logged or exposed

---

## Functional Requirements

### FR-1: GitHub URL Input
- **Priority:** High
- **Description:** Provide UI component for GitHub repository URL input
- **Details:**
  - Text input field for repository URL
  - Support formats:
    - `https://github.com/owner/repo`
    - `https://github.com/owner/repo.git`
    - `github.com/owner/repo`
  - Real-time URL validation
  - Clear button to reset input

### FR-2: Repository Fetching
- **Priority:** High
- **Description:** Fetch repository contents from GitHub
- **Details:**
  - Use GitHub API for efficient fetching (avoid full clone when possible)
  - Support both public and private repositories
  - Handle authentication with optional GitHub token
  - Fetch only code files (filter by extension)
  - Respect file size limits
  - Handle large repositories gracefully

### FR-3: File Filtering
- **Priority:** High
- **Description:** Filter fetched files to only supported code types
- **Details:**
  - Use existing `supported_extensions` configuration
  - Skip binary files, images, and non-code files
  - Skip common directories: `.git`, `node_modules`, `__pycache__`, `venv`, `.venv`, `dist`, `build`
  - Apply file size limits from configuration
  - Apply batch size limits from configuration

### FR-4: UI Mode Toggle
- **Priority:** High
- **Description:** Allow users to switch between upload and GitHub modes
- **Details:**
  - Radio buttons or tabs for mode selection
  - Clear visual distinction between modes
  - Preserve mode selection in session state
  - Clear previous selection when switching modes

### FR-5: File Preview
- **Priority:** Medium
- **Description:** Display fetched files before analysis
- **Details:**
  - Show file list with metadata
  - Display validation results
  - Show file count and total size
  - Allow user to proceed or cancel

### FR-6: Error Handling
- **Priority:** High
- **Description:** Comprehensive error handling for GitHub operations
- **Details:**
  - Validate URL format
  - Handle network errors
  - Handle API rate limiting
  - Handle authentication errors
  - Handle repository not found
  - Provide actionable error messages

### FR-7: Progress Feedback
- **Priority:** Medium
- **Description:** Show progress during repository fetching
- **Details:**
  - Display spinner during fetch operation
  - Show status messages (e.g., "Fetching repository...", "Filtering files...")
  - Display progress for large repositories
  - Allow cancellation of fetch operation

---

## Non-Functional Requirements

### NFR-1: Performance
- Repository fetching should complete within 30 seconds for typical repositories
- Support repositories with up to 1000 code files
- Efficient API usage to minimize rate limiting

### NFR-2: Security
- GitHub tokens must be stored securely in session state only
- Tokens must not be logged or exposed in error messages
- Use HTTPS for all GitHub API requests
- Validate and sanitize all user inputs

### NFR-3: Usability
- Clear instructions for both upload and GitHub modes
- Intuitive UI with minimal learning curve
- Consistent with existing dashboard design
- Responsive feedback for all operations

### NFR-4: Reliability
- Graceful degradation when GitHub API is unavailable
- Retry logic for transient network errors
- Clear error messages for all failure scenarios
- No data loss when switching between modes

### NFR-5: Maintainability
- Modular code structure
- Comprehensive error handling
- Logging for debugging
- Unit tests for all components

---

## Technical Constraints

### TC-1: GitHub API Rate Limits
- Unauthenticated: 60 requests per hour per IP
- Authenticated: 5000 requests per hour per token
- Must implement rate limit detection and handling

### TC-2: File Size Limits
- Respect existing `max_file_size_mb` configuration (10 MB per file)
- Respect existing `max_files_per_batch` configuration (50 files)

### TC-3: Supported File Types
- Use existing `supported_extensions` configuration
- `.py`, `.js`, `.ts`, `.java`, `.cpp`, `.c`, `.go`, `.rs`, `.rb`, `.php`

### TC-4: Dependencies
- Use `requests` library for GitHub API calls (already in requirements)
- Avoid adding heavy dependencies like `gitpython` if possible
- Use GitHub API v3 (REST API)

---

## Out of Scope

The following are explicitly out of scope for this initial implementation:

1. **Git History Analysis:** No analysis of commit history, branches, or git metadata
2. **Subdirectory Selection:** No ability to select specific subdirectories (fetch entire repo)
3. **Branch Selection:** Only fetch default branch (main/master)
4. **GitLab/Bitbucket Support:** Only GitHub repositories supported
5. **Repository Caching:** No persistent caching of fetched repositories
6. **Diff Analysis:** No comparison between repository versions
7. **Pull Request Analysis:** No analysis of specific PRs or commits

---

## Success Metrics

1. **Adoption Rate:** 50%+ of users use GitHub URL instead of manual upload
2. **Success Rate:** 95%+ of valid GitHub URLs successfully fetch and analyze
3. **Performance:** Average fetch time < 10 seconds for typical repositories
4. **Error Rate:** < 5% of fetch attempts result in errors
5. **User Satisfaction:** Positive feedback on ease of use

---

## Dependencies

### Internal Dependencies
- `src/code_analysis_pipeline/upload/file_upload_handler.py` - Integration point
- `src/code_analysis_pipeline/dashboard/components.py` - UI components
- `src/code_analysis_pipeline/config.py` - Configuration
- `src/code_analysis_pipeline/utils/file_utils.py` - File validation utilities

### External Dependencies
- GitHub API v3 (REST API)
- `requests` library (already in requirements)
- Internet connectivity

---

## Implementation Phases

### Phase 1: Core Functionality (MVP)
- GitHub URL input and validation
- Public repository fetching via GitHub API
- File filtering and validation
- Integration with existing upload workflow
- Basic error handling

### Phase 2: Enhanced Features
- GitHub token support for private repositories
- Progress feedback and cancellation
- Rate limit handling and display
- Enhanced error messages

### Phase 3: Optimization
- Performance improvements
- Better file filtering
- UI/UX enhancements
- Comprehensive testing

---

## Risk Assessment

### High Risk
- **GitHub API Rate Limiting:** Could block users from fetching repositories
  - **Mitigation:** Implement rate limit detection, provide clear feedback, support authentication

### Medium Risk
- **Large Repositories:** Could timeout or exceed file limits
  - **Mitigation:** Implement file count/size limits, provide clear error messages
  
- **Network Errors:** Could fail to fetch repositories
  - **Mitigation:** Implement retry logic, provide clear error messages

### Low Risk
- **URL Parsing:** Could fail to parse some GitHub URL formats
  - **Mitigation:** Support multiple URL formats, provide validation feedback

---

## Open Questions

1. **Q:** Should we support fetching specific branches?
   **A:** No, out of scope for initial implementation. Fetch default branch only.

2. **Q:** Should we cache fetched repositories?
   **A:** No, out of scope for initial implementation. Fetch fresh each time.

3. **Q:** Should we support GitLab or Bitbucket?
   **A:** No, GitHub only for initial implementation.

4. **Q:** How should we handle very large repositories (1000+ files)?
   **A:** Apply existing batch size limits, provide clear error message if exceeded.

5. **Q:** Should we allow users to select specific files from the repository?
   **A:** No, out of scope. Fetch all supported code files.

---

## Acceptance Testing Scenarios

### Scenario 1: Successful Public Repository Fetch
1. User selects "GitHub Repository" mode
2. User enters valid public repository URL
3. System validates URL
4. System fetches repository contents
5. System displays list of fetched code files
6. User confirms and starts analysis
7. Analysis completes successfully

### Scenario 2: Invalid URL Handling
1. User enters invalid GitHub URL
2. System displays validation error
3. User corrects URL
4. System validates successfully
5. Fetch proceeds

### Scenario 3: Repository Not Found
1. User enters URL for non-existent repository
2. System attempts fetch
3. System displays "Repository not found" error
4. System suggests checking URL
5. User can retry with different URL

### Scenario 4: Rate Limit Exceeded
1. User attempts fetch after many requests
2. System detects rate limit
3. System displays rate limit error with reset time
4. System suggests using GitHub token
5. User can provide token or wait

### Scenario 5: Large Repository Handling
1. User enters URL for large repository
2. System fetches files
3. System detects file count exceeds limit
4. System displays error with file count
5. System suggests analyzing smaller repository

---

## Documentation Requirements

1. **User Documentation:**
   - How to use GitHub URL feature
   - Supported URL formats
   - How to get GitHub token
   - Troubleshooting common errors

2. **Developer Documentation:**
   - GitHub API integration details
   - File filtering logic
   - Error handling patterns
   - Testing guidelines

3. **Configuration Documentation:**
   - GitHub-related configuration options
   - Rate limit settings
   - File filtering rules

---

## Version History

- **v1.0** (2026-02-07): Initial requirements document
