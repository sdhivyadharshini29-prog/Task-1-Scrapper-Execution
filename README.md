# Task-1-Scrapper-Execution
This project is a lightweight web scraper that extracts structured company information from a publicly accessible company website.  The tool takes  input of two websites and produces a clean, structured JSON output containing company identity, business summary, evidence signals, contact details, hiring signals, and metadata.
The focus is on truthful, resilient extraction, not aggressive crawling.
# Objective 
Convert an ambiguous company website into structured, decision usable data.
Handle different site structures gracefully.
Avoid hallucination and clearly distinguish between found and not found.
Respect scraping limits and public-access rules.
# Features
Extracts:
Company name, website, tagline
Business summary (from About / Company page)
Key pages (About, Products, Careers, Contact, etc.)
Social media links
Emails and phone numbers (if publicly available)
Hiring signals (careers page)
Crawls maximum 15 pages
Handles errors, redirects, and missing pages safely
Outputs structured JSON
Logs pages visited and limitations transparently
# Tech Stack
Python 3
requests
beautifulsoup4
# Output
A file named output.json is generated in the same directory
# Demo Runs
The scraper was tested on real company websites:
Novartis = novartis output.json
Pfizer = pfizer output.json
These files demonstrate consistent structured extraction across different site layouts.
