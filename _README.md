## NOTICE
Hello, welcome to my project! This is a personal endeavor I have been working on during my free time. As this is my first project, you might find some unnecessary elements and mistakes, but I am trying to continuously learn and improve the bugs, which may appear.

Also, I could not publicly share the original git repository as it contains sensitive information such as API keys for SendGrid, email addresses, and folder paths, which are visible throughout the commit history. To protect this private information, I decided to create a new repository where private information is not shown, so neither are commits. Additionally, in the original repository, I used SQL views which would not make sense for other people, so I have replaced them with full SQL scripts to ensure completeness.

## OVERVIEW
This script performs an Extract, Transform, Load (ETL) process to scrape rental offers from the website "bezrealitky.cz". The script extracts data from multiple pages, processes and transforms the data, and loads it into a PostgreSQL database. Additionally, it exports the data to an Excel file and sends an email notification with the latest offers.

## PREREQUISITES
- SendGrid account for email notifications
- PostgreSQL
- Python 3.8.6
- Python packages:
  - beautifulsoup4
  - requests
  - psycopg2
  - pandas
  - numpy
  - openpyxl
  - xlsxwriter
  - sendgrid

## SCRIPT WORKFLOW
- `func_extract()`: Scrapes data from the specified website and stores them.
- `func_get_num_of_pages(url)`: Determines the number of pages to scrape.
- `func_get_offers_from_page(page_url)`: Extracts offer blocks (HTML blocks with usable values) from a page.
- `func_extract_offer_details(offer, page)`: Extracts details from each offer HTML block.
- `func_transformation()`: Processes and transforms the extracted data and converts them to the Pandas DataFrame.
- `func_load()`: Loads transformed data to PostgreSQL.
- `func_excel_export()`: Pulls data from PostgreSQL (together with newly loaded data) with SQL script. Data are converted and transformed to an Excel file which is then exported to the OneDrive folder.
- `func_send_email_notification()`: Sends an email with summary and offers which are cheaper by more than 1% and link to the OneDrive folder.

## COLUMNS
- **id_offer**: Unique identifier for every offer, including inactive ones.
- **min_timestamp**: The first time the offer was spotted.
- **first_load**: Indicates if `min_timestamp` is equal to the first load. If yes, the history for this offer starts on 2024-06-21, and we don’t have any deeper history.
- **max_timestamp**: The last time the offer was spotted.
- **time_on_market**: How long the offer has been on the market (0 -> new offer).
- **active_offer**: If `max_timestamp` equals the last load of offers, the offer is still active.
- **min_total_price**: Minimum price during the entire history of the offer (total price = price_czk + fee_czk).
- **max_total_price**: Maximum price during the entire history of the offer (total price = price_czk + fee_czk).
- **min_max_total_price_diff**: Difference between `min_total_price` and `max_total_price`.
- **current_price**: Current price (without fees).
- **current_fee**: Current fees.
- **current_total_price**: Current price plus fees.
- **total_price_per_m2**: Total price per square meter (excluding terrace, cellar, balcony).
- **district**: Prague district where the flat is located (or a nearby city/village, e.g., Říčany).
- **district_proximity**: Area of the district (this column was created manually, so it can contain blank values).
- **disposition**: Disposition of the flat.
- **size_m2**: Size of the flat (excluding terrace, cellar, balcony).
- **total_price_per_disp_diff**: The difference between the average price per disposition and the offer. For example, a 1+kk flat with a value of -50 means that the total price of the offer is 50% cheaper than the average price for the 1+kk disposition (outliers removed).
- **total_price_m2_per_district_diff**: The difference between the average price per square meter in a given district and the offer. For example, a flat in Holešovice with a value of 50 means that the total price of the offer is 50% more expensive than the average price per square meter in the Holešovice district (outliers removed).
- **z_score_total_price_per_m2**: Z-score calculated from `total_price_per_m2` column. This column is used to detect outliers and remove them from counting average.
- **street**: Street where the flat is located.
- **prague_region**: Indicates whether the flat is located in Prague (Y) or outside of Prague (N, e.g., Říčany, Hostivice).
- **mhd_minutes_away**: Walking time in minutes to the nearest MHD stop (metro, tram, bus).
- **balcony_size_m2**: Size of the balcony, if the flat has one.
- **cellar_size_m2**: Size of the cellar, if the flat has one.
- **terrace_size_m2**: Size of the terrace, if the flat has one.
- **parking_or_and_garage**: Indicates if the flat has a parking space and/or garage.
- **elevator**: Indicates if the flat has an elevator.
- **partly_or_fully_equiped**: Indicates if the flat is partly or fully equipped.
- **href**: Hyperlink to the flat offer.
