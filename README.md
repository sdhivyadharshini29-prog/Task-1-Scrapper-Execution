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

# Demo one
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin
from datetime import datetime
import json


IMPORTANT_KEYWORDS = {
    "identity": ["about", "company", "team"],
    "offerings": ["product", "solution", "service", "pricing", "features"],
    "evidence": ["client", "case-study", "testimonial", "award", "partner"],
    "hiring": ["career", "jobs", "hiring", "join-us"],
    "contact": ["contact", "support", "location"]
}

SOCIAL_DOMAINS = [
    "linkedin.com", "twitter.com", "x.com",
    "facebook.com", "instagram.com", "youtube.com", "github.com"
]

MAX_PAGES = 15


# SCRAPER CLASS

class BusinessScraper:
    def __init__(self, base_url):
        self.base_url = base_url.rstrip("/")
        self.data = {
            "identity": {
                "company_name": None,
                "website": self.base_url,
                "tagline": None
            },
            "business_summary": {
                "what_they_do": None,
                "primary_offerings": [],
                "target_segments": []
            },
            "evidence": {
                "key_pages": {},
                "social_links": [],
                "signals": []
            },
            "contact_location": {
                "emails": [],
                "phones": [],
                "address": None,
                "contact_url": None
            },
            "hiring": {
                "careers_url": None,
                "roles_mentioned": []
            },
            "metadata": {
                "timestamp": str(datetime.now()),
                "pages_visited": [],
                "errors": [],
                "limitations": "JS-heavy content may not be fully rendered. Best-effort HTML scraping."
            }
        }

    # Fetch page with limit
    def fetch(self, url):
        if len(self.data["metadata"]["pages_visited"]) >= MAX_PAGES:
            self.data["metadata"]["errors"].append(
                f"Page limit reached ({MAX_PAGES}). Stopping crawl."
            )
            return None

        try:
            self.data["metadata"]["pages_visited"].append(url)
            response = requests.get(
                url,
                timeout=10,
                headers={"User-Agent": "Mozilla/5.0"}
            )
            response.raise_for_status()
            return BeautifulSoup(response.text, "html.parser")

        except Exception as e:
            self.data["metadata"]["errors"].append(
                f"Error fetching {url}: {str(e)}"
            )
            return None

    # Main scrape logic
    def scrape(self):
        soup = self.fetch(self.base_url)
        if not soup:
            return self.data

        # Identity
        if soup.title:
            self.data["identity"]["company_name"] = soup.title.text.strip()

        meta_desc = soup.find("meta", attrs={"name": "description"})
        if meta_desc:
            self.data["identity"]["tagline"] = meta_desc.get("content")

        # Links scanning
        for link in soup.find_all("a", href=True):
            href = urljoin(self.base_url, link["href"])
            text = link.text.lower().strip()

            # Social links
            if any(domain in href for domain in SOCIAL_DOMAINS):
                if href not in self.data["evidence"]["social_links"]:
                    self.data["evidence"]["social_links"].append(href)

            # Emails & phones
            if "mailto:" in href:
                self.data["contact_location"]["emails"].append(
                    href.replace("mailto:", "")
                )

            if "tel:" in href:
                self.data["contact_location"]["phones"].append(
                    href.replace("tel:", "")
                )

            # Categorize pages
            for category, keywords in IMPORTANT_KEYWORDS.items():
                if any(k in href.lower() or k in text for k in keywords):
                    if category not in self.data["evidence"]["key_pages"]:
                        self.data["evidence"]["key_pages"][category] = href

        # Business summary from About / Company page
        about_url = self.data["evidence"]["key_pages"].get("identity")
        if about_url:
            about_soup = self.fetch(about_url)
            if about_soup:
                paragraphs = [
                    p.get_text().strip()
                    for p in about_soup.find_all("p")
                    if len(p.get_text()) > 50
                ]
                if paragraphs:
                    self.data["business_summary"]["what_they_do"] = " ".join(
                        paragraphs[:2]
                    )

        # Final links
        self.data["contact_location"]["contact_url"] = (
            self.data["evidence"]["key_pages"].get("contact")
        )
        self.data["hiring"]["careers_url"] = (
            self.data["evidence"]["key_pages"].get("hiring")
        )

        return self.data


#  ENTRY POINT

if __name__ == "__main__":
    target_url = input("Enter company website URL (include https://): ").strip()

    scraper = BusinessScraper(target_url)
    result = scraper.scrape()

    with open("output.json", "w") as f:
        json.dump(result, f, indent=4)

    print("\n Scraping complete")
    print(" Output saved as output.json")
    print("\n--- STRUCTURED COMPANY INFO ---")
    print(json.dumps(result, indent=4))

!python scraper.py

!mv output.json novartis_output.json

!ls

# Demo two
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin
from datetime import datetime
import json


IMPORTANT_KEYWORDS = {
    "identity": ["about", "company", "team"],
    "offerings": ["product", "solution", "service", "pricing", "features"],
    "evidence": ["client", "case-study", "testimonial", "award", "partner"],
    "hiring": ["career", "jobs", "hiring", "join-us"],
    "contact": ["contact", "support", "location"]
}

SOCIAL_DOMAINS = [
    "linkedin.com", "twitter.com", "x.com",
    "facebook.com", "instagram.com", "youtube.com", "github.com"
]

MAX_PAGES = 15


# SCRAPER CLASS

class BusinessScraper:
    def __init__(self, base_url):
        self.base_url = base_url.rstrip("/")
        self.data = {
            "identity": {
                "company_name": None,
                "website": self.base_url,
                "tagline": None
            },
            "business_summary": {
                "what_they_do": None,
                "primary_offerings": [],
                "target_segments": []
            },
            "evidence": {
                "key_pages": {},
                "social_links": [],
                "signals": []
            },
            "contact_location": {
                "emails": [],
                "phones": [],
                "address": None,
                "contact_url": None
            },
            "hiring": {
                "careers_url": None,
                "roles_mentioned": []
            },
            "metadata": {
                "timestamp": str(datetime.now()),
                "pages_visited": [],
                "errors": [],
                "limitations": "JS-heavy content may not be fully rendered. Best-effort HTML scraping."
            }
        }

    # Fetch page with limit
    def fetch(self, url):
        if len(self.data["metadata"]["pages_visited"]) >= MAX_PAGES:
            self.data["metadata"]["errors"].append(
                f"Page limit reached ({MAX_PAGES}). Stopping crawl."
            )
            return None

        try:
            self.data["metadata"]["pages_visited"].append(url)
            response = requests.get(
                url,
                timeout=10,
                headers={"User-Agent": "Mozilla/5.0"}
            )
            response.raise_for_status()
            return BeautifulSoup(response.text, "html.parser")

        except Exception as e:
            self.data["metadata"]["errors"].append(
                f"Error fetching {url}: {str(e)}"
            )
            return None

    # Main scrape logic
    def scrape(self):
        soup = self.fetch(self.base_url)
        if not soup:
            return self.data

        # Identity
        if soup.title:
            self.data["identity"]["company_name"] = soup.title.text.strip()

        meta_desc = soup.find("meta", attrs={"name": "description"})
        if meta_desc:
            self.data["identity"]["tagline"] = meta_desc.get("content")

        # Links scanning
        for link in soup.find_all("a", href=True):
            href = urljoin(self.base_url, link["href"])
            text = link.text.lower().strip()

            # Social links
            if any(domain in href for domain in SOCIAL_DOMAINS):
                if href not in self.data["evidence"]["social_links"]:
                    self.data["evidence"]["social_links"].append(href)

            # Emails & phones
            if "mailto:" in href:
                self.data["contact_location"]["emails"].append(
                    href.replace("mailto:", "")
                )

            if "tel:" in href:
                self.data["contact_location"]["phones"].append(
                    href.replace("tel:", "")
                )

            # Categorize pages
            for category, keywords in IMPORTANT_KEYWORDS.items():
                if any(k in href.lower() or k in text for k in keywords):
                    if category not in self.data["evidence"]["key_pages"]:
                        self.data["evidence"]["key_pages"][category] = href

        # Business summary from About / Company page
        about_url = self.data["evidence"]["key_pages"].get("identity")
        if about_url:
            about_soup = self.fetch(about_url)
            if about_soup:
                paragraphs = [
                    p.get_text().strip()
                    for p in about_soup.find_all("p")
                    if len(p.get_text()) > 50
                ]
                if paragraphs:
                    self.data["business_summary"]["what_they_do"] = " ".join(
                        paragraphs[:2]
                    )

        # Final links
        self.data["contact_location"]["contact_url"] = (
            self.data["evidence"]["key_pages"].get("contact")
        )
        self.data["hiring"]["careers_url"] = (
            self.data["evidence"]["key_pages"].get("hiring")
        )

        return self.data


#  ENTRY POINT

if __name__ == "__main__":
    target_url = input("Enter company website URL (include https://): ").strip()

    scraper = BusinessScraper(target_url)
    result = scraper.scrape()

    with open("output.json", "w") as f:
        json.dump(result, f, indent=4)

    print("\n Scraping complete")
    print(" Output saved as output.json")
    print("\n--- STRUCTURED COMPANY INFO ---")
    print(json.dumps(result, indent=4))

!ls

!mv output.json pfizer_output.json

!ls


These files demonstrate consistent structured extraction across different site layouts.
