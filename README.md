# dns
import socket
import whois
import dns.resolver
import os

# Function to validate and sanitize the input URL
def sanitize_url(url):
    if url.startswith("http://") or url.startswith("https://"):
        url = url.split("://")[1]
    return url.split("/")[0]  # Remove path if present

# Function to get the IP address of a website
def get_ip_address(url):
    try:
        return socket.gethostbyname(url)
    except socket.gaierror:
        return "Unable to resolve IP address."

# Function to get DNS records of a website
def get_dns_records(url):
    records = {}
    try:
        for record_type in ["A", "AAAA", "MX", "NS", "TXT"]:
            answers = dns.resolver.resolve(url, record_type, raise_on_no_answer=False)
            records[record_type] = [r.to_text() for r in answers]
    except Exception as e:
        records['Error'] = str(e)
    return records

# Function to get WHOIS information of a website
def get_whois_info(url):
    try:
        domain_info = whois.whois(url)
        return domain_info
    except Exception as e:
        return f"Unable to retrieve WHOIS information: {e}"

# Main function
def main():
    url = input("Enter the website URL (e.g., example.com): ").strip()
    url = sanitize_url(url)

    # Check if the 'whois' command is available (Linux-specific)
    if os.system("which whois > /dev/null 2>&1") != 0:
        print("Error: 'whois' command not found. Install it using 'sudo apt install whois'.")
        return

    # Get IP address
    print("\\nFetching IP address...")
    ip_address = get_ip_address(url)
    print(f"IP Address: {ip_address}")

    # Get DNS records
    print("\\nFetching DNS records...")
    dns_records = get_dns_records(url)
    for record_type, values in dns_records.items():
        print(f"{record_type} Records:")
        if values:
            for value in values:
                print(f"  - {value}")
        else:
            print("  - None")

    # Get WHOIS information
    print("\\nFetching WHOIS information...")
    whois_info = get_whois_info(url)
    if isinstance(whois_info, str):
        print(whois_info)
    else:
        for key, value in whois_info.items():
            print(f"{key}: {value}")

if __name__ == "__main__":
    main()
