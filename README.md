import requests
from bs4 import BeautifulSoup
import re
import matplotlib.pyplot as plt

def convert_price_to_float(price):
    # Remove currency symbols and commas, and then convert to float
    price = re.sub(r'[^\d.]', '', price)
    return float(price) if price else 0.0


def get_amazon_products(search_query):

    base_url = 'https://www.amazon.in'

    headers = {
        'User-Agent': 'Mozilla/4.0'
    }

    search_query = search_query.replace(' ', '+')

    url = f'{base_url}/s?k={search_query}'

    response = requests.get(url, headers=headers)

    products_data = []

    if response.status_code == 200:

        soup = BeautifulSoup(response.content, 'html.parser')

        products = soup.find_all(
            'div',
            {'data-component-type': 's-search-result'}
        )

        for product in products:

            title = product.h2.text.strip() if product.h2 else 'N/A'

            price_whole = product.find(
                'span',
                class_='a-price-whole'
            )

            if price_whole:
                price = price_whole.text.strip()
            else:
                price = '0'

            products_data.append({
                'Product': title,
                'Price': price
            })

    return sorted(
        products_data,
        key=lambda x: convert_price_to_float(x['Price'])
    )


search_query = input('Enter product to search on Amazon: ')

products = get_amazon_products(search_query)

# Display product names and prices
if products:

    print("\nProducts Found:\n")

    for i, product in enumerate(products, start=1):

        print(f"{i}. Product Name : {product['Product']}")
        print(f"   Price        : ₹{product['Price']}")
        print("-----------------------------------")

    # Displaying product data using a bar chart
    product_names = [
        product['Product'][:30]
        if len(product['Product']) > 30
        else product['Product']
        for product in products
    ]

    product_prices = [
        convert_price_to_float(product['Price'])
        for product in products
    ]

    plt.figure(figsize=(10, 6))

    plt.barh(
        range(len(product_prices)),
        product_prices,
        color='skyblue'
    )

    plt.xlabel('Price')
    plt.ylabel('Product')

    plt.title(
        f'Products and their Prices on Amazon for '
        f'{search_query.capitalize()} (Ascending Order)'
    )

    plt.yticks(
        range(len(product_prices)),
        product_names
    )

    plt.tight_layout()
    plt.show()

else:
    print('No products found.')
