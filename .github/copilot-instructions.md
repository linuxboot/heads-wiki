# Heads Wiki - Jekyll Documentation Site

**Always follow these instructions first and only fall back to additional search and context gathering if the information here is incomplete or found to be in error.**

The Heads Wiki is a Jekyll-based documentation site for the Heads firmware project. It uses the just-the-docs theme with a dark color scheme and is hosted on GitHub Pages.

## Working Effectively

### Bootstrap and Build the Site
- Install Ruby and dependencies:
  ```bash
  sudo apt-get update && sudo apt-get install -y ruby-full build-essential zlib1g-dev
  ```
- Install Jekyll and required gems:
  ```bash
  sudo gem install bundler jekyll just-the-docs
  ```
- Build the site:
  ```bash
  jekyll build --config local_config.yml
  ```
  - Build takes approximately 1.4 seconds. NEVER CANCEL builds.
  - Set timeout to 60+ seconds for build commands as a safety margin.

### Run the Site Locally
- Serve for local development:
  ```bash
  jekyll serve --config local_config.yml --host 0.0.0.0 --port 4000
  ```
- Access at: `http://localhost:4000`
- The site will auto-regenerate when files change (unless using `--detach`)
- To stop the server: Press Ctrl+C or run `pkill -f jekyll`

### Clean Build
- Remove build artifacts:
  ```bash
  rm -rf _site .jekyll-cache
  ```
- Always clean before fresh builds to ensure reproducibility

## Validation

### Manual Testing Requirements
- ALWAYS verify the site builds and serves successfully after making changes
- Test that all navigation links work correctly
- Verify markdown rendering appears correct with the just-the-docs theme
- Check that any new pages appear in the site navigation structure
- Test both light and dark themes are working (site uses dark by default)

### Validation Scenarios
After making documentation changes, always:
1. Build the site and confirm no Jekyll errors: `jekyll build --config local_config.yml`
2. Serve the site locally: `jekyll serve --config local_config.yml --host 0.0.0.0 --port 4000`
3. Navigate to modified pages and verify they render correctly
4. Check the site navigation menu includes new pages appropriately
5. Verify all internal links work (no broken markdown links)

## Repository Structure

### Key Directories and Files
```
.
├── .github/           # GitHub configuration (funding, etc.)
├── About/             # About pages
├── Community.md       # Community documentation
├── Development/       # Development guides (7 files)
├── Installing-and-Configuring/  # Installation guides (11 files)
├── PDFs/              # PDF documentation
├── images/            # Image assets
├── index.md           # Homepage
├── _config.yml        # GitHub Pages Jekyll config
├── local_config.yml   # Local development Jekyll config
└── README.md          # Repository README
```

### Configuration Files
- **_config.yml**: GitHub Pages configuration using `remote_theme: just-the-docs/just-the-docs`
- **local_config.yml**: Local development configuration using `theme: "just-the-docs"`
- Both configs use dark color scheme and enable heading anchors

### Content Organization
- 34 total Markdown files across the repository
- Documentation is organized into logical sections with Jekyll front matter
- Uses Jekyll's navigation system with `nav_order`, `parent`, and `grand_parent` properties
- Pages include metadata like `permalink`, `title`, and `has_children`

## Common Tasks

### Adding New Documentation
1. Create new `.md` file in appropriate directory
2. Add Jekyll front matter:
   ```yaml
   ---
   layout: default
   title: Your Page Title
   nav_order: X
   parent: Parent Section (if applicable)
   permalink: /your-url/
   ---
   ```
3. Write content using standard Markdown
4. Build and test locally before committing

### Modifying Existing Documentation
1. Edit the Markdown file directly
2. Maintain existing Jekyll front matter structure
3. Follow the existing style and formatting conventions
4. Test changes locally with Jekyll serve

### Working with Images
- Place images in the `images/` directory
- Reference using: `![Alt text]({{ site.baseurl }}/images/filename.png)`
- Or use full URLs for external images

## Timing Expectations
- **Jekyll build**: ~1.4 seconds (very fast)
- **Jekyll serve startup**: ~1.5 seconds  
- **Dependency installation**: 2-5 minutes for Ruby gems
- **NEVER CANCEL**: Always wait for operations to complete, even though they're fast

## Important Notes

### No CI/Testing Infrastructure
- This repository has NO automated testing or CI workflows
- No linting tools are configured
- No build verification beyond manual testing
- Validation is entirely manual - you must test locally

### Deployment
- Site is automatically deployed to GitHub Pages when changes are pushed to master
- Uses the `_config.yml` configuration for production
- Available at: https://osresearch.net (as configured in CNAME)

### Jekyll Deprecation Warnings
- The just-the-docs theme generates many Sass deprecation warnings
- These warnings are cosmetic and do not affect functionality
- Warnings are expected and normal - do not attempt to fix them

### Theme Customization
- Site uses dark color scheme by default
- Custom styling should be minimal to maintain consistency
- Follow just-the-docs theme conventions for navigation and layout

### Development Best Practices
- Always use `local_config.yml` for local development
- Test all changes locally before pushing
- Maintain the existing Jekyll front matter structure
- Keep navigation hierarchy logical and organized
- Verify internal links are correct after moving/renaming files