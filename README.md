
# Growth For Impact - Web Scraping Assignment

## Overview
This project implements a multi-stage web scraping pipeline to extract comprehensive job posting information from climate-focused companies. Starting with only company names and descriptions, the system progressively gathers online presence data, career pages, and detailed job listings.

---

## Methodology

### Stage 1: Website & LinkedIn URL Extraction

**Objective:** Find official company websites and LinkedIn profiles using company name and description.

**Implementation:** `WebsiteFinderSelenium` class

**Process:**
1. **Search Query Construction**
   - Extracts 2-3 key descriptive words from company description
   - Filters out common stop words (the, a, and, etc.)
   - Creates targeted Google search queries combining company name with keywords
   - Example: `"Polestar driving society uncompromised official website"`

2. **Intelligent Web Search**
   - Uses Selenium WebDriver to perform real Google searches
   - Implements anti-detection measures:
     - Custom user agents
     - Disabled automation flags
     - Realistic browser behavior with delays
   - Separate searches for website and LinkedIn URLs

3. **Result Extraction**
   - Parses multiple HTML elements (cite tags, result links, search links)
   - Validates URLs by checking accessibility
   - Attempts alternative formats (www prefix) if initial validation fails

4. **Progressive Processing**
   - Batch processing (8 companies at a time) to avoid CAPTCHA
   - Saves progress every 3 companies
   - 8-second delays between searches to mimic human behavior
   - Resumes from last checkpoint if interrupted

**Key Features:**
- Context-aware searching using company descriptions
- Robust URL validation and normalization
- Session persistence for interrupted runs
- Rate limiting to avoid detection

---

### Stage 2: Career Page & Job Listings URL Extraction

**Objective:** Locate career pages and job listing portals from company websites.

**Implementation:** `CareerPageFinder` class

**Process:**
1. **Website Crawling**
   - Navigates to company website using Selenium
   - Extracts all links from the page
   - Handles cookie consent popups automatically

2. **Multi-Tier Detection System**

   **Priority 1 - Known Job Platforms (Score: 10)**
   - Detects 20+ ATS platforms: Lever, Greenhouse, Workable, Ashby, BambooHR, etc.
   - Automatically identifies embedded job boards

   **Priority 2 - Strong Career Keywords in URL (Score: 6-8)**
   - URL patterns: `/careers`, `/jobs`, `/join-us`, `/hiring`, `/work-with-us`
   - Cross-validates with link text content
   
   **Priority 3 - Link Text Analysis (Score: 5)**
   - Searches for phrases: "career", "jobs", "join us", "work with us", "opportunities"

3. **Job Listings Page Discovery**
   - Follows redirect links with phrases like "view jobs", "open positions"
   - Detects embedded iframes containing job boards
   - Checks for job listing indicators in page content:
     - "apply now", "job title", "job description", "open position"
   - Attempts `/jobs` suffix on job platform URLs
   - Falls back to career page if job-specific page not found

4. **URL Filtering**
   - Excludes individual job postings (patterns like `/job/1234`, `/apply/123`)
   - Filters out non-career pages (privacy, terms, blog, news)
   - Validates all URLs before saving

**Key Features:**
- Hierarchical scoring system for link relevance
- Intelligent redirect following
- Iframe detection for embedded job boards
- Content-based heuristics for validation
- Common URL pattern testing

---

### Stage 3: Job Details Extraction from LinkedIn

**Objective:** Scrape detailed job information from LinkedIn company pages.

**Implementation:** Main scraping function with Selenium

**Process:**
1. **Authentication & Session Management**
   - One-time manual LinkedIn login
   - Session cookies saved in `linkedin_profile` directory
   - Automatic session reuse for subsequent runs
   - User-data-dir profile persistence

2. **Jobs Page Navigation**
   - Constructs jobs URL: `{linkedin_company_url}/jobs/?lang=en`
   - Forces English language for consistent selectors
   - Waits for job card elements to load (10-second timeout)
   - Performs scroll actions to trigger lazy-loaded content

3. **Job Card Iteration**
   - Identifies job cards using CSS selector: `li[class*='job-card-container']`
   - Processes top 3 jobs per company
   - Re-finds job list in each iteration to prevent stale element errors
   - Clicks each job card to load details in right pane

4. **Detailed Information Extraction**
   
   For each job, extracts:
   
   **Job Title & URL**
   - Selector: `h1[class*='t-24']`
   - Extracts embedded link using `urljoin` for absolute URLs
   
   **Location & Date Posted**
   - Selector: `div[class*='job-details-jobs-unified-top-card__tertiary-description']`
   - Parses text split by '·' separator
   
   **Job Description**
   - Selector: `#job-details`
   - Clicks "Show more" button if present: `button[class*='description__show-more-less-button'][aria-expanded='false']`
   - Extracts full expanded text using `innerText` attribute

5. **Data Storage Structure**
   ```
   job post1 title, job post1 URL, job post1 location, job post1 date, job post1 description
   job post2 title, job post2 URL, job post2 location, job post2 date, job post2 description
   job post3 title, job post3 URL, job post3 location, job post3 date, job post3 description
   ```

6. **Error Handling**
   - `StaleElementReferenceException`: Re-finds elements and continues
   - `TimeoutException`: Skips company if jobs page doesn't load
   - Missing elements: Records "Not Found" for specific fields
   - Recovers from errors with scroll actions and wait periods

**Key Features:**
- Session persistence eliminates repeated logins
- Robust element re-finding strategy
- Dynamic content expansion (Show more buttons)
- Comprehensive error recovery
- Batch processing with progress preservation

---

## Technical Architecture

### Technologies Used
- **Python 3.12**
- **Selenium WebDriver** - Browser automation
- **Pandas** - Data manipulation and CSV handling
- **BeautifulSoup4** - HTML parsing (auxiliary)
- **webdriver-manager** - Automatic ChromeDriver management
- **Requests** - URL validation

### Anti-Detection Measures
- Randomized user agents
- Disabled automation flags
- CDP commands to hide webdriver property
- Human-like delays (3-8 seconds between actions)
- Batch processing to avoid rate limits
- Session persistence to reduce suspicious activity

### Data Persistence Strategy
- **Single Output File Approach**: All batches update the same CSV
- **Incremental Updates**: Only processes missing data
- **Checkpoint System**: Saves every 3 companies
- **Resume Capability**: Continues from `START_INDEX` parameter
- **Data Validation**: Checks for existing data before scraping

```

---

## Results Summary


### Data Quality Features
- Validated URLs (accessibility checked)
- Cleaned and normalized data
- Comprehensive error handling
- "Not Found" placeholders for missing data
- UTF-8 encoding support
