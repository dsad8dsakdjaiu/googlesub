import argparse
from serpapi import GoogleSearch
import time

# Function to perform a search query
def search_google(query, api_key):
    params = {
        "q": query,
        "hl": "en",
        "gl": "us",
        "google_domain": "google.com",
        "api_key": api_key,
        "num": 10  # Number of results per request (adjustable)
    }
    search = GoogleSearch(params)
    results = search.get_dict()
    return results

# Function to extract URLs from search results
def extract_urls(search_results):
    urls = []
    if "organic_results" in search_results:
        for result in search_results["organic_results"]:
            urls.append(result["link"])
    return urls

# Function to extract subdomain from a URL
def extract_subdomain(url):
    parts = url.split('//')[-1].split('.')
    if len(parts) >= 3:
        return parts[0]
    return None

# Parse command-line arguments
parser = argparse.ArgumentParser(description="Subdomain enumeration using SerpApi")
parser.add_argument("-u", "--url", required=True, help="Base URL for subdomain enumeration (e.g., instagram.com)")
parser.add_argument("-t", "--token", required=True, help="SerpApi API token")
args = parser.parse_args()

# Initialize variables
api_key = args.token
base_domain = args.url
base_query = f"site:*.{base_domain} -www.{base_domain}"
subdomains = set()
max_iterations = 5  # Set the number of iterations

# Perform the initial search
results = search_google(base_query, api_key)
urls = extract_urls(results)
new_subdomains = {extract_subdomain(url) for url in urls if extract_subdomain(url)}
subdomains.update(new_subdomains)

# Loop for a fixed number of iterations
for i in range(max_iterations):
    if not new_subdomains:
        break  # Exit loop if no new subdomains are found

    exclude_query = " ".join([f"-{sub}" for sub in subdomains])
    query = f"site:*.{base_domain} -www.{base_domain} {exclude_query}"

    results = search_google(query, api_key)
    urls = extract_urls(results)
    new_subdomains = {extract_subdomain(url) for url in urls if extract_subdomain(url)}

    if not new_subdomains:
        break  # Exit loop if no new subdomains are found

    subdomains.update(new_subdomains)

    # Pause to avoid hitting rate limits
    time.sleep(2)

# Print all unique subdomains
for subdomain in sorted(subdomains):
    print(subdomain)