import csv
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin

# Function to scrape product details from the listing page
def scrape_listing_page(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    products = []
    listings = soup.find_all('div', {'data-component-type': 's-search-result'})

    for listing in listings:
        product = {}
        
        # Extract product URL
        url_element = listing.find('a', {'class': 'a-link-normal s-no-outline'})
        if url_element:
            product['URL'] = urljoin(url, url_element['href'])
        
        # Extract product name
        name_element = listing.find('span', {'class': 'a-size-base-plus a-color-base a-text-normal'})
        if name_element:
            product['Name'] = name_element.text
        
        # Extract product price
        price_element = listing.find('span', {'class': 'a-price-whole'})
        if price_element:
            product['Price'] = price_element.text
        
        # Extract product rating
        rating_element = listing.find('span', {'class': 'a-icon-alt'})
        if rating_element:
            product['Rating'] = rating_element.text.split()[0]
        
        # Extract number of reviews
        reviews_element = listing.find('span', {'class': 'a-size-base'})
        if reviews_element:
            product['Reviews'] = reviews_element.text.split()[0]
        
        products.append(product)

    return products

# Function to scrape product details from the individual product page
def scrape_product_page(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    product_info = {}

    # Extract description
    description_element = soup.find('div', {'id': 'productDescription'})
    if description_element:
        product_info['Description'] = description_element.text.strip()

    # Extract ASIN
    asin_element = soup.find('th', text='ASIN')
    if asin_element:
        product_info['ASIN'] = asin_element.find_next_sibling('td').text.strip()

    # Extract product description
    prod_desc_element = soup.find('div', {'id': 'productDescription'})
    if prod_desc_element:
        product_info['ProductDescription'] = prod_desc_element.text.strip()

    # Extract manufacturer
    manufacturer_element = soup.find('th', text='Manufacturer')
    if manufacturer_element:
        product_info['Manufacturer'] = manufacturer_element.find_next_sibling('td').text.strip()

    return product_info

# Scrape the listing pages and collect product URLs
base_url = 'https://www.amazon.in/s?k=bags&crid=2M096C6104MLT&qid=1653308124&sprefix=ba,aps%2C283&ref=sr_pg'
product_urls = []

for page_num in range(1, 21):
    url = base_url + str(page_num)
    products = scrape_listing_page(url)
    for product in products:
        product_urls.append(product['URL'])

# Scrape individual product pages and collect product details
product_details = []

for url in product_urls[:200]:
    details = scrape_product_page(url)
    product_details.append(details)

# Save the data to a CSV file
filename = 'amazon_products.csv'
fieldnames = ['URL', 'Name', 'Price', 'Rating', 'Reviews', 'Description', 'ASIN', 'ProductDescription', 'Manufacturer']

with open(filename, 'w', newline='',delimiter=',', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(product_details)

print("Scraping and data export completed!")
