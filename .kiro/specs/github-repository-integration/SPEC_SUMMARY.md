# GitHub Repository Integration - Specification Summary

## Overview

This specification defines the requirements for adding GitHub repository URL support to the Code Analysis Pipeline dashboard, allowing users to analyze code directly from GitHub repositories without manual file uploads.

---

## Key Features

### 1. **GitHub URL Input**
- Users can paste a GitHub repository URL instead of uploading files
- Supports multiple URL formats (https://github.com/owner/repo, github.com/owner/repo, etc.)
- Real-time URL validation

### 2. **Dual Input Modes**
- Toggle between "Upload Files" and "GitHub Repository" modes
- Clear visual distinction between modes
- Seamless integration with existing workflow

### 3. **Smart File Fetching**
- Uses GitHub API for efficient fetching (no full clone needed)
- Automatically filters to supported code file types
- Skips common non-code directories (node_modules, .git, etc.)
- Respects existing file size and batch limits

### 4. **Public & Private Repository Support**
- Public repositories work without authentication
- Optional GitHub token for private repositories
- Secure token handling (session-only storage)

### 5. **Comprehensive Error Handling**
- Invalid URL detection
- Repository not found (404)
- Network errors
- Rate limiting detection
- Authentication errors
- Clear, actionable error messages

---

## User Experience Flow

```
1. User selects "GitHub Repository" mode
   ↓
2. User enters repository URL
   ↓
3. System validates URL format
   ↓
4. System fetches repository via GitHub API
   ↓
5. System filters to supported code files
   ↓
6. System displays fetched files for review
   ↓
7. User confirms and starts analysis
   ↓
8. Analysis proceeds with existing workflow
```

---

## Technical Approach

### Architecture
```
Dashboard UI (components.py)
    ↓
GitHub Fetcher Module (NEW)
    ↓
GitHub API Client (NEW)
    ↓
File Upload Handler (EXISTING - Enhanced)
    ↓
Existing Analysis Pipeline
```

### Key Components

1. **`GitHubFetcher`** (New Module)
   - URL validation and parsing
   - GitHub API integration
   - File filtering logic
   - Error handling

2. **`GitHubAPIClient`** (New Module)
   - GitHub API v3 REST calls
   - Authentication handling
   - Rate limit detection
   - Repository content fetching

3. **Enhanced `FileUploadHandler`**
   - Add GitHub mode support
   - Integrate with GitHubFetcher
   - Unified validation workflow

4. **Enhanced Dashboard Components**
   - Mode toggle UI
   - GitHub URL input field
   - Optional token input
   - Fetched files preview

---

## Implementation Plan

### Phase 1: Core Functionality (MVP)
**Estimated Time:** 4-6 hours

**Tasks:**
1. Create `GitHubFetcher` module
2. Create `GitHubAPIClient` module
3. Add URL validation logic
4. Implement public repository fetching
5. Add file filtering
6. Update `FileUploadHandler` for GitHub mode
7. Update dashboard UI with mode toggle
8. Basic error handling
9. Unit tests

**Deliverables:**
- Working GitHub URL input
- Public repository fetching
- File filtering and validation
- Integration with existing workflow

### Phase 2: Enhanced Features
**Estimated Time:** 2-3 hours

**Tasks:**
1. Add GitHub token support
2. Implement rate limit handling
3. Add progress feedback
4. Enhanced error messages
5. Cancellation support
6. Additional tests

**Deliverables:**
- Private repository support
- Rate limit detection and feedback
- Better UX with progress indicators

### Phase 3: Polish & Testing
**Estimated Time:** 2-3 hours

**Tasks:**
1. Performance optimization
2. UI/UX improvements
3. Comprehensive testing
4. Documentation
5. Error scenario testing

**Deliverables:**
- Production-ready feature
- Complete test coverage
- User documentation

**Total Estimated Time:** 8-12 hours

---

## Files to Create

### New Files
1. `src/code_analysis_pipeline/github/github_fetcher.py`
   - Main GitHub fetching logic
   - URL validation
   - File filtering

2. `src/code_analysis_pipeline/github/github_api_client.py`
   - GitHub API integration
   - Authentication
   - Rate limit handling

3. `src/code_analysis_pipeline/github/__init__.py`
   - Module exports

4. `tests/test_github_fetcher.py`
   - Unit tests for GitHubFetcher

5. `tests/test_github_api_client.py`
   - Unit tests for GitHubAPIClient

### Files to Modify
1. `src/code_analysis_pipeline/upload/file_upload_handler.py`
   - Add GitHub mode support
   - Integrate GitHubFetcher

2. `src/code_analysis_pipeline/dashboard/components.py`
   - Add mode toggle UI
   - Add GitHub URL input
   - Add token input (optional)

3. `src/code_analysis_pipeline/config.py`
   - Add GitHub configuration section

4. `.env.example`
   - Add GITHUB_TOKEN example

5. `README.md`
   - Document GitHub URL feature

6. `DEPLOYMENT.md`
   - Add GitHub token setup instructions

---

## Configuration

### New Configuration Options

```python
class GitHubConfig(BaseModel):
    """GitHub integration configuration."""
    
    # API Settings
    api_base_url: str = "https://api.github.com"
    api_version: str = "2022-11-28"
    
    # Authentication (optional)
    github_token: str = Field(default_factory=lambda: os.getenv("GITHUB_TOKEN", ""))
    
    # Rate Limiting
    rate_limit_buffer: int = 10  # Reserve 10 requests
    check_rate_limit: bool = True
    
    # Fetching Settings
    max_files_to_fetch: int = 50  # Align with max_files_per_batch
    max_file_size_bytes: int = 10 * 1024 * 1024  # 10 MB
    timeout_seconds: int = 30
    
    # Filtering
    skip_directories: List[str] = Field(default_factory=lambda: [
        ".git", "node_modules", "__pycache__", "venv", ".venv",
        "dist", "build", "target", ".idea", ".vscode"
    ])
    
    # Retry Settings
    max_retries: int = 3
    retry_delay_seconds: int = 2
```

---

## API Usage

### GitHub API Endpoints Used

1. **Get Repository Contents**
   ```
   GET /repos/{owner}/{repo}/contents/{path}
   ```
   - Fetch directory contents recursively
   - Filter to code files only

2. **Get File Content**
   ```
   GET /repos/{owner}/{repo}/contents/{file_path}
   ```
   - Fetch individual file content
   - Base64 decode content

3. **Get Rate Limit Status**
   ```
   GET /rate_limit
   ```
   - Check remaining API calls
   - Get reset time

### Rate Limits
- **Unauthenticated:** 60 requests/hour per IP
- **Authenticated:** 5,000 requests/hour per token

---

## Error Scenarios & Handling

| Error | Detection | User Message | Suggested Action |
|-------|-----------|--------------|------------------|
| Invalid URL | Regex validation | "Invalid GitHub URL format" | "Use format: https://github.com/owner/repo" |
| Repo Not Found | 404 response | "Repository not found" | "Check URL and repository visibility" |
| Rate Limited | 403 with rate limit header | "GitHub API rate limit exceeded" | "Wait {time} or provide GitHub token" |
| Network Error | Connection timeout | "Network error fetching repository" | "Check internet connection and retry" |
| Auth Required | 404 on private repo | "Repository is private" | "Provide GitHub token in settings" |
| Too Many Files | File count > limit | "Repository has too many files" | "Repository has {count} files, limit is {limit}" |
| File Too Large | File size > limit | "Some files exceed size limit" | "Files over {limit}MB will be skipped" |

---

## Security Considerations

1. **Token Storage**
   - Store in session state only (not persisted)
   - Never log token values
   - Clear on session end

2. **Input Validation**
   - Validate all URLs before processing
   - Sanitize repository owner/name
   - Prevent path traversal attacks

3. **API Security**
   - Use HTTPS only
   - Validate API responses
   - Handle malicious content gracefully

4. **Rate Limiting**
   - Respect GitHub's rate limits
   - Implement exponential backoff
   - Provide clear feedback to users

---

## Testing Strategy

### Unit Tests
- URL validation logic
- GitHub API client methods
- File filtering logic
- Error handling paths

### Integration Tests
- End-to-end repository fetching
- Dashboard UI integration
- Error scenario handling

### Manual Testing Scenarios
1. Public repository (small)
2. Public repository (large)
3. Private repository with token
4. Invalid URLs
5. Non-existent repositories
6. Rate limit scenarios
7. Network error scenarios

---

## Success Criteria

✅ **Feature Complete When:**
1. Users can input GitHub repository URLs
2. System fetches and filters code files
3. Fetched files integrate with existing workflow
4. All error scenarios handled gracefully
5. Public repositories work without authentication
6. Private repositories work with token
7. Rate limiting detected and communicated
8. All tests pass
9. Documentation complete

---

## Next Steps

1. **Review Requirements** - Confirm requirements with stakeholders
2. **Create Design Document** - Detail technical design and architecture
3. **Create Tasks** - Break down into specific implementation tasks
4. **Begin Implementation** - Start with Phase 1 (MVP)

---

## Questions for Review

1. **Scope Confirmation:**
   - Is GitHub-only support acceptable? (No GitLab/Bitbucket)
   - Is default branch only acceptable? (No branch selection)
   - Is full repository fetch acceptable? (No subdirectory selection)

2. **Technical Decisions:**
   - Should we use GitHub API or git clone? (Recommendation: API for efficiency)
   - Should we cache fetched repositories? (Recommendation: No for MVP)
   - Should we support very large repos (1000+ files)? (Recommendation: Apply limits)

3. **UX Decisions:**
   - Should GitHub mode be default or upload mode? (Recommendation: Upload default)
   - Should we show file tree or flat list? (Recommendation: Flat list for MVP)
   - Should we allow file deselection? (Recommendation: No for MVP)

---

## Document Status

- **Status:** Draft - Ready for Review
- **Version:** 1.0
- **Date:** 2026-02-07
- **Author:** Kiro AI Assistant
- **Next Review:** After stakeholder feedback
