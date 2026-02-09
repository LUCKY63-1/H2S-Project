# GitHub Repository Integration - Design Document

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Streamlit Dashboard                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Dashboard Components (components.py)          │  │
│  │  ┌────────────────┐      ┌────────────────────────┐  │  │
│  │  │  Upload Mode   │      │  GitHub Repo Mode      │  │  │
│  │  │  (Existing)    │      │  (New)                 │  │  │
│  │  └────────┬───────┘      └──────────┬─────────────┘  │  │
│  └───────────┼────────────────────────┼────────────────┘  │
└──────────────┼────────────────────────┼───────────────────┘
               │                        │
               ▼                        ▼
    ┌──────────────────┐    ┌──────────────────────┐
    │ FileUploadHandler│◄───┤   GitHubFetcher      │
    │   (Enhanced)     │    │   (New Module)       │
    └────────┬─────────┘    └──────────┬───────────┘
             │                          │
             │                          ▼
             │              ┌──────────────────────┐
             │              │  GitHubAPIClient     │
             │              │  (New Module)        │
             │              └──────────┬───────────┘
             │                         │
             │                         ▼
             │              ┌──────────────────────┐
             │              │   GitHub API v3      │
             │              │   (External)         │
             │              └──────────────────────┘
             │
             ▼
    ┌──────────────────────┐
    │  File Validator      │
    │  (Existing)          │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │  Analysis Pipeline   │
    │  (Existing)          │
    └──────────────────────┘
```

### 1.2 Component Responsibilities

#### 1.2.1 Dashboard Components (Enhanced)
**File:** `src/code_analysis_pipeline/dashboard/components.py`

**Responsibilities:**
- Render mode toggle UI (Upload vs GitHub)
- Render GitHub URL input field
- Render optional GitHub token input
- Display fetched files preview
- Handle mode switching
- Coordinate with FileUploadHandler

**New Functions:**
- `render_input_mode_selector()` - Toggle between upload and GitHub modes
- `render_github_input_section()` - GitHub URL and token inputs
- `render_fetched_files_preview()` - Display fetched files

#### 1.2.2 FileUploadHandler (Enhanced)
**File:** `src/code_analysis_pipeline/upload/file_upload_handler.py`

**Responsibilities:**
- Support both upload and GitHub modes
- Integrate with GitHubFetcher
- Unified file validation workflow
- Convert GitHub files to UploadedFile format

**New Methods:**
- `render_github_interface()` - Render GitHub input UI
- `fetch_from_github(url, token)` - Fetch files from GitHub
- `convert_github_files(files)` - Convert to UploadedFile format

#### 1.2.3 GitHubFetcher (New)
**File:** `src/code_analysis_pipeline/github/github_fetcher.py`

**Responsibilities:**
- Parse and validate GitHub URLs
- Coordinate file fetching via GitHubAPIClient
- Filter files by extension and size
- Skip excluded directories
- Handle errors and provide user feedback

**Key Methods:**
- `parse_github_url(url)` - Extract owner and repo
- `validate_url(url)` - Validate URL format
- `fetch_repository(owner, repo, token)` - Fetch all code files
- `filter_files(files)` - Apply filtering rules
- `get_file_metadata(files)` - Generate metadata

#### 1.2.4 GitHubAPIClient (New)
**File:** `src/code_analysis_pipeline/github/github_api_client.py`

**Responsibilities:**
- Make GitHub API requests
- Handle authentication
- Detect and handle rate limiting
- Retry logic for transient errors
- Parse API responses

**Key Methods:**
- `get_repository_contents(owner, repo, path)` - Get directory contents
- `get_file_content(owner, repo, path)` - Get file content
- `check_rate_limit()` - Check remaining API calls
- `is_rate_limited()` - Detect rate limit errors

---

## 2. Data Models

### 2.1 GitHubRepository

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class GitHubRepository:
    """Represents a GitHub repository."""
    owner: str
    name: str
    url: str
    default_branch: str = "main"
    is_private: bool = False
```

### 2.2 GitHubFile

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class GitHubFile:
    """Represents a file from GitHub."""
    name: str
    path: str
    size: int
    content: bytes
    sha: str
    download_url: str
    encoding: str = "utf-8"
```

### 2.3 FetchResult

```python
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class FetchResult:
    """Result of fetching from GitHub."""
    success: bool
    files: List[GitHubFile]
    total_files: int
    filtered_files: int
    skipped_files: int
    errors: List[str]
    warnings: List[str]
    rate_limit_remaining: Optional[int] = None
    rate_limit_reset: Optional[int] = None
```

---

## 3. API Integration

### 3.1 GitHub API Endpoints

#### 3.1.1 Get Repository Contents
```
GET /repos/{owner}/{repo}/contents/{path}
```

**Headers:**
```
Accept: application/vnd.github.v3+json
Authorization: Bearer {token}  # Optional
```

**Response:**
```json
[
  {
    "name": "file.py",
    "path": "src/file.py",
    "size": 1024,
    "type": "file",
    "sha": "abc123",
    "download_url": "https://raw.githubusercontent.com/...",
    "content": "base64_encoded_content",
    "encoding": "base64"
  }
]
```

#### 3.1.2 Get Rate Limit
```
GET /rate_limit
```

**Response:**
```json
{
  "resources": {
    "core": {
      "limit": 5000,
      "remaining": 4999,
      "reset": 1372700873
    }
  }
}
```

### 3.2 Rate Limit Handling

**Strategy:**
1. Check rate limit before fetching
2. Reserve buffer (10 requests)
3. If remaining < buffer, show warning
4. If rate limited (403), parse reset time
5. Display clear message to user

**Implementation:**
```python
def check_rate_limit(self) -> Tuple[bool, Optional[str]]:
    """Check if we have enough API calls remaining.
    
    Returns:
        Tuple of (can_proceed, error_message)
    """
    response = self._make_request("/rate_limit")
    remaining = response["resources"]["core"]["remaining"]
    reset_time = response["resources"]["core"]["reset"]
    
    if remaining < self.rate_limit_buffer:
        reset_dt = datetime.fromtimestamp(reset_time)
        return False, f"Rate limit low ({remaining} remaining). Resets at {reset_dt}"
    
    return True, None
```

---

## 4. File Filtering Logic

### 4.1 Filtering Rules

```python
class FileFilter:
    """Filter files based on rules."""
    
    def should_include_file(self, file_path: str, file_size: int) -> Tuple[bool, Optional[str]]:
        """Determine if file should be included.
        
        Returns:
            Tuple of (include, reason_if_excluded)
        """
        # Check extension
        if not self._has_supported_extension(file_path):
            return False, "Unsupported file type"
        
        # Check size
        if file_size > self.max_file_size:
            return False, f"File too large ({file_size} bytes)"
        
        # Check if in excluded directory
        if self._in_excluded_directory(file_path):
            return False, "In excluded directory"
        
        return True, None
    
    def _has_supported_extension(self, file_path: str) -> bool:
        """Check if file has supported extension."""
        ext = Path(file_path).suffix.lower()
        return ext in self.supported_extensions
    
    def _in_excluded_directory(self, file_path: str) -> bool:
        """Check if file is in excluded directory."""
        path_parts = Path(file_path).parts
        return any(excluded in path_parts for excluded in self.excluded_dirs)
```

### 4.2 Excluded Directories

Default excluded directories:
- `.git` - Git metadata
- `node_modules` - Node.js dependencies
- `__pycache__` - Python cache
- `venv`, `.venv` - Python virtual environments
- `dist`, `build` - Build outputs
- `target` - Rust/Java build output
- `.idea`, `.vscode` - IDE settings

---

## 5. Error Handling

### 5.1 Error Types

```python
class GitHubError(Exception):
    """Base exception for GitHub operations."""
    pass

class InvalidURLError(GitHubError):
    """Invalid GitHub URL format."""
    pass

class RepositoryNotFoundError(GitHubError):
    """Repository not found or not accessible."""
    pass

class RateLimitError(GitHubError):
    """GitHub API rate limit exceeded."""
    def __init__(self, reset_time: int):
        self.reset_time = reset_time
        super().__init__(f"Rate limit exceeded. Resets at {reset_time}")

class AuthenticationError(GitHubError):
    """Authentication required or failed."""
    pass

class NetworkError(GitHubError):
    """Network error during GitHub operation."""
    pass

class FileLimitExceededError(GitHubError):
    """Too many files in repository."""
    def __init__(self, count: int, limit: int):
        self.count = count
        self.limit = limit
        super().__init__(f"Repository has {count} files, limit is {limit}")
```

### 5.2 Error Handling Flow

```python
def fetch_repository(self, url: str, token: Optional[str] = None) -> FetchResult:
    """Fetch repository with comprehensive error handling."""
    try:
        # Parse URL
        owner, repo = self.parse_github_url(url)
        
        # Check rate limit
        can_proceed, error = self.api_client.check_rate_limit()
        if not can_proceed:
            raise RateLimitError(error)
        
        # Fetch files
        files = self._fetch_all_files(owner, repo, token)
        
        # Filter files
        filtered = self._filter_files(files)
        
        # Check file count
        if len(filtered) > self.max_files:
            raise FileLimitExceededError(len(filtered), self.max_files)
        
        return FetchResult(
            success=True,
            files=filtered,
            total_files=len(files),
            filtered_files=len(filtered),
            skipped_files=len(files) - len(filtered),
            errors=[],
            warnings=[]
        )
        
    except InvalidURLError as e:
        return FetchResult(
            success=False,
            files=[],
            total_files=0,
            filtered_files=0,
            skipped_files=0,
            errors=[f"Invalid URL: {str(e)}"],
            warnings=[]
        )
    
    except RepositoryNotFoundError:
        return FetchResult(
            success=False,
            files=[],
            total_files=0,
            filtered_files=0,
            skipped_files=0,
            errors=["Repository not found. Check URL and repository visibility."],
            warnings=[]
        )
    
    # ... handle other exceptions
```

---

## 6. UI Design

### 6.1 Mode Selection

```python
def render_input_mode_selector():
    """Render mode selection UI."""
    st.subheader("📤 Code Input Method")
    
    mode = st.radio(
        "Choose how to provide your code:",
        options=["Upload Files", "GitHub Repository"],
        horizontal=True,
        key="input_mode"
    )
    
    # Clear previous selection when switching modes
    if "previous_mode" in st.session_state and st.session_state.previous_mode != mode:
        if mode == "Upload Files":
            st.session_state.github_url = ""
            st.session_state.github_files = []
        else:
            st.session_state.uploaded_files = []
    
    st.session_state.previous_mode = mode
    
    return mode
```

### 6.2 GitHub Input Section

```python
def render_github_input_section():
    """Render GitHub URL input section."""
    st.markdown("""
    **Enter a GitHub repository URL to analyze:**
    
    Supported formats:
    - `https://github.com/owner/repo`
    - `github.com/owner/repo`
    """)
    
    # URL input
    github_url = st.text_input(
        "GitHub Repository URL",
        placeholder="https://github.com/owner/repo",
        key="github_url_input",
        help="Enter the URL of a public or private GitHub repository"
    )
    
    # Optional token input (collapsed by default)
    with st.expander("🔑 Private Repository? (Optional)", expanded=False):
        st.markdown("""
        For private repositories, provide a GitHub Personal Access Token.
        
        **How to get a token:**
        1. Go to GitHub Settings → Developer settings → Personal access tokens
        2. Generate new token (classic)
        3. Select 'repo' scope
        4. Copy the token
        """)
        
        github_token = st.text_input(
            "GitHub Token",
            type="password",
            key="github_token_input",
            help="Optional: Required for private repositories"
        )
        
        if github_token:
            st.session_state.github_token = github_token
            st.success("✅ Token configured")
    
    return github_url
```

### 6.3 Fetch Button and Progress

```python
def render_fetch_button(github_url: str):
    """Render fetch button with progress feedback."""
    if not github_url:
        st.info("👆 Enter a GitHub repository URL above")
        return None
    
    if st.button("🔍 Fetch Repository", type="primary", use_container_width=True):
        # Validate URL first
        if not GitHubFetcher.validate_url(github_url):
            st.error("❌ Invalid GitHub URL format")
            return None
        
        # Show progress
        with st.spinner("🔄 Fetching repository from GitHub..."):
            progress_bar = st.progress(0)
            status_text = st.empty()
            
            # Fetch files
            status_text.text("Connecting to GitHub API...")
            progress_bar.progress(20)
            
            token = st.session_state.get("github_token")
            fetcher = GitHubFetcher()
            
            status_text.text("Fetching repository contents...")
            progress_bar.progress(40)
            
            result = fetcher.fetch_repository(github_url, token)
            
            status_text.text("Filtering code files...")
            progress_bar.progress(60)
            
            if result.success:
                status_text.text("Validating files...")
                progress_bar.progress(80)
                
                # Convert to UploadedFile format
                uploaded_files = fetcher.convert_to_uploaded_files(result.files)
                
                progress_bar.progress(100)
                status_text.text("✅ Fetch complete!")
                
                return uploaded_files
            else:
                st.error(f"❌ Failed to fetch repository")
                for error in result.errors:
                    st.error(f"• {error}")
                return None
    
    return None
```

---

## 7. Configuration

### 7.1 GitHub Configuration Class

```python
class GitHubConfig(BaseModel):
    """GitHub integration configuration."""
    
    # API Settings
    api_base_url: str = "https://api.github.com"
    api_version: str = "2022-11-28"
    
    # Authentication
    github_token: str = Field(default_factory=lambda: os.getenv("GITHUB_TOKEN", ""))
    
    # Rate Limiting
    rate_limit_buffer: int = 10
    check_rate_limit: bool = True
    
    # Fetching Settings
    max_files_to_fetch: int = 50
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
    retry_backoff_multiplier: float = 2.0
```

### 7.2 Integration with Main Config

```python
class Config(BaseModel):
    """Main configuration class."""
    
    dspy: DSPyConfig = Field(default_factory=DSPyConfig)
    multi_provider: MultiProviderConfig = Field(default_factory=MultiProviderConfig)
    file_processing: FileProcessingConfig = Field(default_factory=FileProcessingConfig)
    streamlit: StreamlitConfig = Field(default_factory=StreamlitConfig)
    logging: LoggingConfig = Field(default_factory=LoggingConfig)
    cache: CacheConfig = Field(default_factory=CacheConfig)
    performance: PerformanceConfig = Field(default_factory=PerformanceConfig)
    github: GitHubConfig = Field(default_factory=GitHubConfig)  # NEW
```

---

## 8. Security Considerations

### 8.1 Token Security

**Requirements:**
1. Store tokens only in session state (not persisted)
2. Never log token values
3. Clear tokens on session end
4. Use HTTPS for all API requests
5. Validate token format before use

**Implementation:**
```python
def store_github_token(token: str) -> bool:
    """Store GitHub token securely.
    
    Args:
        token: GitHub personal access token
        
    Returns:
        True if token is valid format
    """
    # Validate token format (ghp_... or github_pat_...)
    if not (token.startswith("ghp_") or token.startswith("github_pat_")):
        logger.warning("Invalid GitHub token format")
        return False
    
    # Store in session state only
    st.session_state.github_token = token
    
    # Never log the actual token
    logger.info("GitHub token configured (length: %d)", len(token))
    
    return True
```

### 8.2 Input Validation

**URL Validation:**
```python
def validate_url(url: str) -> bool:
    """Validate GitHub URL format.
    
    Args:
        url: GitHub repository URL
        
    Returns:
        True if valid format
    """
    # Supported patterns
    patterns = [
        r"^https?://github\.com/[\w-]+/[\w.-]+/?$",
        r"^github\.com/[\w-]+/[\w.-]+/?$",
        r"^https?://github\.com/[\w-]+/[\w.-]+\.git$"
    ]
    
    # Remove trailing slashes and .git
    url = url.rstrip("/").rstrip(".git")
    
    # Check against patterns
    return any(re.match(pattern, url) for pattern in patterns)
```

**Path Sanitization:**
```python
def sanitize_path(path: str) -> str:
    """Sanitize file path to prevent traversal attacks.
    
    Args:
        path: File path from GitHub
        
    Returns:
        Sanitized path
    """
    # Remove leading slashes
    path = path.lstrip("/")
    
    # Remove parent directory references
    path = path.replace("../", "").replace("..\\", "")
    
    # Normalize path
    path = os.path.normpath(path)
    
    return path
```

---

## 9. Performance Optimization

### 9.1 Parallel File Fetching

```python
async def fetch_files_parallel(self, file_paths: List[str]) -> List[GitHubFile]:
    """Fetch multiple files in parallel.
    
    Args:
        file_paths: List of file paths to fetch
        
    Returns:
        List of fetched files
    """
    async with aiohttp.ClientSession() as session:
        tasks = [
            self._fetch_file_async(session, path)
            for path in file_paths
        ]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Filter out errors
        files = [r for r in results if isinstance(r, GitHubFile)]
        
        return files
```

### 9.2 Caching Strategy

**Session-level caching:**
```python
@st.cache_data(ttl=3600)  # Cache for 1 hour
def fetch_repository_cached(url: str, token: Optional[str] = None) -> FetchResult:
    """Fetch repository with caching.
    
    Args:
        url: GitHub repository URL
        token: Optional GitHub token
        
    Returns:
        Fetch result
    """
    fetcher = GitHubFetcher()
    return fetcher.fetch_repository(url, token)
```

---

## 10. Testing Strategy

### 10.1 Unit Tests

**Test Coverage:**
- URL parsing and validation
- File filtering logic
- Error handling
- Rate limit detection
- Token validation

**Example Test:**
```python
def test_url_validation():
    """Test GitHub URL validation."""
    fetcher = GitHubFetcher()
    
    # Valid URLs
    assert fetcher.validate_url("https://github.com/owner/repo")
    assert fetcher.validate_url("github.com/owner/repo")
    assert fetcher.validate_url("https://github.com/owner/repo.git")
    
    # Invalid URLs
    assert not fetcher.validate_url("https://gitlab.com/owner/repo")
    assert not fetcher.validate_url("not-a-url")
    assert not fetcher.validate_url("")
```

### 10.2 Integration Tests

**Test Scenarios:**
1. Fetch small public repository
2. Fetch repository with mixed file types
3. Handle repository not found
4. Handle rate limiting
5. Handle network errors

**Example Test:**
```python
@pytest.mark.integration
def test_fetch_public_repository():
    """Test fetching a real public repository."""
    fetcher = GitHubFetcher()
    
    # Use a small, stable test repository
    url = "https://github.com/octocat/Hello-World"
    
    result = fetcher.fetch_repository(url)
    
    assert result.success
    assert len(result.files) > 0
    assert result.errors == []
```

### 10.3 Mock Testing

**Mock GitHub API:**
```python
@pytest.fixture
def mock_github_api(monkeypatch):
    """Mock GitHub API responses."""
    def mock_get(*args, **kwargs):
        class MockResponse:
            status_code = 200
            def json(self):
                return [
                    {
                        "name": "test.py",
                        "path": "test.py",
                        "size": 100,
                        "type": "file",
                        "download_url": "https://example.com/test.py",
                        "content": base64.b64encode(b"print('hello')").decode()
                    }
                ]
        return MockResponse()
    
    monkeypatch.setattr("requests.get", mock_get)
```

---

## 11. Deployment Considerations

### 11.1 Environment Variables

```bash
# .env.example
GITHUB_TOKEN=ghp_your_token_here  # Optional: For private repos
```

### 11.2 Dependencies

**New Dependencies:**
- `requests` - Already in requirements.txt
- `aiohttp` - For async file fetching (optional optimization)

**No new dependencies required for MVP!**

---

## 12. Monitoring and Logging

### 12.1 Logging Strategy

```python
# Log levels
logger.info("Fetching repository: %s", url)  # User actions
logger.debug("API request: GET %s", endpoint)  # API calls
logger.warning("Rate limit low: %d remaining", remaining)  # Warnings
logger.error("Failed to fetch: %s", error)  # Errors

# Never log sensitive data
logger.info("Token configured (length: %d)", len(token))  # Good
logger.info("Token: %s", token)  # BAD - Never do this!
```

### 12.2 Metrics to Track

- Fetch success rate
- Average fetch time
- Files fetched per repository
- Rate limit hits
- Error types and frequency

---

## 13. Future Enhancements

### 13.1 Phase 2 Features
- Branch selection
- Subdirectory selection
- File deselection UI
- Repository caching
- Progress cancellation

### 13.2 Phase 3 Features
- GitLab/Bitbucket support
- Commit/PR analysis
- Diff analysis
- Git history analysis

---

## Document Status

- **Version:** 1.0
- **Status:** Ready for Implementation
- **Date:** 2026-02-07
- **Next Step:** Create tasks.md
