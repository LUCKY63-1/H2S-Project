# GitHub-DSPy Integration Testing Guide

**Date**: 2026-02-07  
**Status**: Ready for Testing  
**Quick Fix**: ✅ IMPLEMENTED

---

## 🎯 Overview

The quick fix has been implemented to connect GitHub-fetched files with the DSPy analysis pipeline. This guide will help you test the integration end-to-end.

---

## 🚀 Quick Start

### Step 1: Restart Dashboard

```bash
# Run the restart script (recommended)
restart_dashboard.bat

# OR manually:
# 1. Stop old process
taskkill /F /IM streamlit.exe

# 2. Wait 2 seconds
timeout /t 2

# 3. Start fresh
python run_dashboard.py
```

### Step 2: Hard Refresh Browser

When the browser opens at `http://localhost:8503`:

1. **Press `Ctrl + Shift + R`** (Chrome/Edge) or `Ctrl + F5` (Firefox)
2. This forces a complete reload and clears cached blank pages
3. You should see the dashboard with "🔍 Code Analysis Pipeline" header

### Step 3: Verify Dashboard Loads

You should see:
- ✅ Header: "🔍 Code Analysis Pipeline"
- ✅ Sidebar with configuration options
- ✅ Main area with "📤 Code Input Method" selector
- ✅ Two options: "📁 Upload Files" and "🔗 GitHub Repository"

---

## 🧪 Test Suite

### Test 1: GitHub Fetch → Analysis (CRITICAL)

**Purpose**: Verify GitHub files flow through DSPy pipeline

**Steps**:
1. Click "🔗 GitHub Repository" in the input method selector
2. Enter URL: `https://github.com/octocat/Hello-World`
3. Click "🔍 Fetch Repository"
4. Wait for fetch to complete (should take 5-10 seconds)
5. Verify: Files are displayed with validation results
6. Click "🚀 Start Analysis" button
7. Verify: Progress bar appears and updates
8. Verify: Analysis completes and results appear in tabs
9. Verify: All 4 tabs have content:
   - 📄 File Summaries
   - 🔍 Function Explanations
   - 🔄 Project Flow
   - ❓ Q&A Pairs
10. Click "💾 Export Results"
11. Verify: JSON file downloads successfully

**Expected Results**:
- ✅ Fetch completes without errors
- ✅ Files are validated (should show 1-3 Python files)
- ✅ Analysis runs and shows progress
- ✅ Results appear in all tabs
- ✅ Export works correctly

**If This Fails**:
- Check browser console (F12) for errors
- Check terminal for error messages
- Verify API keys are set in `.env` file
- Try a different repository

---

### Test 2: Upload → Analysis (Regression Test)

**Purpose**: Verify upload functionality still works

**Steps**:
1. Click "📁 Upload Files" in the input method selector
2. Upload 1-3 Python files from your computer
3. Verify: Files are validated
4. Click "🚀 Start Analysis"
5. Verify: Analysis runs correctly
6. Verify: Results appear as expected

**Expected Results**:
- ✅ Upload works as before
- ✅ Validation works
- ✅ Analysis runs
- ✅ Results are correct

**If This Fails**:
- The quick fix may have broken upload functionality
- Report the issue immediately

---

### Test 3: Mode Switching

**Purpose**: Verify clean state management

**Steps**:
1. Start in "📁 Upload Files" mode
2. Upload 2-3 files
3. Verify: Files are displayed
4. Switch to "🔗 GitHub Repository" mode
5. Verify: Upload files are cleared
6. Enter GitHub URL and fetch repository
7. Verify: GitHub files are displayed
8. Switch back to "📁 Upload Files" mode
9. Verify: GitHub files are cleared
10. Upload new files
11. Verify: Analysis works correctly

**Expected Results**:
- ✅ Files are cleared when switching modes
- ✅ No leftover state from previous mode
- ✅ Analysis works in both modes

**If This Fails**:
- Session state cleanup may not be working
- Try refreshing the page (F5)

---

### Test 4: Error Handling

**Purpose**: Verify error scenarios are handled gracefully

**Test 4.1: Invalid URL**
```
Steps:
1. Enter invalid URL: "not-a-url"
2. Click "Fetch Repository"
3. Verify: Clear error message appears
```

**Test 4.2: Non-existent Repository**
```
Steps:
1. Enter URL: "https://github.com/nonexistent/repo-that-does-not-exist"
2. Click "Fetch Repository"
3. Verify: 404 error message appears
```

**Test 4.3: Private Repository (No Token)**
```
Steps:
1. Enter URL to a private repository
2. Don't enter token
3. Click "Fetch Repository"
4. Verify: Authentication error appears
```

**Test 4.4: Rate Limiting**
```
Steps:
1. Fetch multiple repositories quickly (5-10 times)
2. Verify: Rate limit message appears if limit is hit
3. Verify: Message suggests using a token
```

**Expected Results**:
- ✅ All errors show clear, helpful messages
- ✅ No crashes or blank screens
- ✅ User can recover and try again

---

### Test 5: Large Repository

**Purpose**: Verify file limit enforcement

**Steps**:
1. Enter URL to a large repository (e.g., `https://github.com/django/django`)
2. Click "Fetch Repository"
3. Verify: Fetch completes
4. Verify: File limit message appears (max 100 files)
5. Verify: Only first 100 files are processed
6. Click "Start Analysis"
7. Verify: Analysis runs on the limited set

**Expected Results**:
- ✅ File limit is enforced
- ✅ Clear message about file limit
- ✅ Analysis works on limited set

---

### Test 6: Progress Tracking

**Purpose**: Verify progress updates are accurate

**Steps**:
1. Fetch a repository with 5-10 files
2. Click "Start Analysis"
3. Watch the progress bar
4. Verify: Progress updates smoothly (0% → 100%)
5. Verify: Status messages update for each stage
6. Verify: No freezing or hanging

**Expected Results**:
- ✅ Progress bar updates smoothly
- ✅ Status messages are clear
- ✅ No UI freezing

---

### Test 7: Results Display

**Purpose**: Verify results are displayed correctly

**Steps**:
1. Complete a GitHub analysis
2. Check each tab:
   - **File Summaries**: Should show summary for each file
   - **Function Explanations**: Should show functions with explanations
   - **Project Flow**: Should show project structure and flow
   - **Q&A Pairs**: Should show questions and answers
3. Verify: All content is readable and formatted correctly
4. Verify: No missing data or errors

**Expected Results**:
- ✅ All tabs have content
- ✅ Content is well-formatted
- ✅ No missing or corrupted data

---

### Test 8: Export Functionality

**Purpose**: Verify export works with GitHub files

**Steps**:
1. Complete a GitHub analysis
2. Click "💾 Export Results"
3. Verify: JSON file downloads
4. Open the JSON file
5. Verify: Contains all results
6. Verify: File paths are correct
7. Verify: GitHub metadata is included (if implemented)

**Expected Results**:
- ✅ Export downloads successfully
- ✅ JSON is valid and complete
- ✅ All data is present

---

## 🐛 Troubleshooting

### Issue: Dashboard Shows Blank Screen

**Solution**:
```bash
# 1. Stop all Streamlit processes
taskkill /F /IM streamlit.exe

# 2. Clear cache
rmdir /s /q .streamlit\cache
rmdir /s /q __pycache__

# 3. Restart
python run_dashboard.py

# 4. Hard refresh browser (Ctrl + Shift + R)
```

### Issue: Port 8503 Already in Use

**Solution**:
```bash
# Find and kill process on port 8503
for /f "tokens=5" %a in ('netstat -ano ^| findstr :8503') do taskkill /F /PID %a

# Then restart
python run_dashboard.py
```

### Issue: API Key Warning

**Solution**:
```bash
# Verify .env file exists and has keys
type .env

# Should show:
# OPENROUTER_API_KEY=sk-or-v1-...
# NVIDIA_API_KEY=nvapi-...

# If missing, add them to .env file
```

### Issue: GitHub Fetch Fails

**Possible Causes**:
1. **Invalid URL**: Check URL format
2. **Private repo**: Add GitHub token
3. **Rate limit**: Wait or add token
4. **Network issue**: Check internet connection

**Solution**:
```bash
# Test GitHub API directly
curl https://api.github.com/repos/octocat/Hello-World

# Should return JSON with repo info
```

### Issue: Analysis Doesn't Start

**Possible Causes**:
1. **No files fetched**: Check fetch completed successfully
2. **Files not in session state**: Check browser console (F12)
3. **API key missing**: Check sidebar for warning

**Solution**:
```bash
# Check browser console (F12) for errors
# Look for messages like:
# - "No files found in session state"
# - "API key not configured"
# - "Error starting analysis"
```

### Issue: Results Don't Appear

**Possible Causes**:
1. **Analysis failed**: Check terminal for errors
2. **API error**: Check API key is valid
3. **Network issue**: Check internet connection

**Solution**:
```bash
# Check terminal output for errors
# Look for messages like:
# - "Error calling OpenRouter API"
# - "Rate limit exceeded"
# - "Invalid API key"
```

---

## 📊 Test Results Template

Use this template to record your test results:

```markdown
## Test Results - [Date]

### Test 1: GitHub Fetch → Analysis
- [ ] Fetch completed successfully
- [ ] Files validated correctly
- [ ] Analysis started
- [ ] Progress tracked
- [ ] Results displayed
- [ ] Export worked
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 2: Upload → Analysis
- [ ] Upload worked
- [ ] Validation worked
- [ ] Analysis ran
- [ ] Results correct
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 3: Mode Switching
- [ ] Files cleared on switch
- [ ] No leftover state
- [ ] Both modes work
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 4: Error Handling
- [ ] Invalid URL handled
- [ ] Non-existent repo handled
- [ ] Private repo handled
- [ ] Rate limit handled
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 5: Large Repository
- [ ] File limit enforced
- [ ] Clear message shown
- [ ] Analysis worked
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 6: Progress Tracking
- [ ] Progress bar updated
- [ ] Status messages clear
- [ ] No freezing
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 7: Results Display
- [ ] All tabs have content
- [ ] Content formatted correctly
- [ ] No missing data
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Test 8: Export Functionality
- [ ] Export downloaded
- [ ] JSON valid
- [ ] All data present
- **Status**: ✅ PASS / ❌ FAIL
- **Notes**: 

### Overall Status
- **Tests Passed**: X/8
- **Tests Failed**: X/8
- **Critical Issues**: 
- **Minor Issues**: 
- **Ready for Production**: ✅ YES / ❌ NO
```

---

## 🎯 Success Criteria

### Minimum Viable (Must Pass)
- ✅ Test 1: GitHub Fetch → Analysis
- ✅ Test 2: Upload → Analysis (regression)
- ✅ Test 3: Mode Switching

### Important (Should Pass)
- ✅ Test 4: Error Handling
- ✅ Test 6: Progress Tracking
- ✅ Test 7: Results Display

### Nice to Have (Can Fail)
- ⚠️ Test 5: Large Repository
- ⚠️ Test 8: Export Functionality

---

## 📝 Reporting Issues

If you find issues, please report with:

1. **Test Number**: Which test failed
2. **Steps**: What you did
3. **Expected**: What should happen
4. **Actual**: What actually happened
5. **Screenshots**: If applicable
6. **Console Logs**: Browser console (F12) and terminal output
7. **Environment**: OS, browser, Python version

**Example**:
```
Test 1 Failed: GitHub Fetch → Analysis

Steps:
1. Entered URL: https://github.com/octocat/Hello-World
2. Clicked "Fetch Repository"
3. Fetch completed successfully
4. Clicked "Start Analysis"
5. Nothing happened

Expected: Analysis should start and show progress

Actual: Button clicked but no response

Console Logs:
- Browser: "No files found in session state"
- Terminal: No errors

Environment:
- OS: Windows 11
- Browser: Chrome 120
- Python: 3.11.5
```

---

## 🚀 Next Steps After Testing

### If All Tests Pass
1. Mark Task 10.3-10.5 as complete
2. Start Task 10.6-10.8 (source tracking)
3. Move to Task 11 (metadata preservation)

### If Tests Fail
1. Document the failures
2. Analyze the root cause
3. Fix the issues
4. Re-test

### If Critical Test Fails (Test 1)
1. **STOP** - This is the core functionality
2. Review the quick fix implementation
3. Check session state keys
4. Verify file flow through pipeline
5. Fix before proceeding

---

## 📚 Related Documents

- **Implementation**: `QUICK_FIX_COMPLETE.md`
- **Analysis**: `GITHUB_DSPY_INTEGRATION_ANALYSIS.md`
- **Status**: `INTEGRATION_STATUS.md`
- **Tasks**: `tasks.md`
- **Troubleshooting**: `FIX_DASHBOARD_ISSUES.md`

---

## ✅ Ready to Test!

The quick fix is implemented and ready for testing. Follow the steps above to verify the integration works correctly.

**Start with Test 1** - this is the critical path that must work!

Good luck! 🚀
