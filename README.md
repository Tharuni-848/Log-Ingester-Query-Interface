# Log-Ingester
from http.server import BaseHTTPRequestHandler, HTTPServer
import json

logs = []

class LogIngestorHandler(BaseHTTPRequestHandler):
    def _send_response(self, status, message):
        self.send_response(status)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({"status": message}).encode('utf-8'))

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        log_data = json.loads(self.rfile.read(content_length))
        logs.append(log_data)
        self._send_response(200, "success")

if __name__ == '__main__':
    server_address = ('', 3000)
    log_ingestor = HTTPServer(server_address, LogIngestorHandler)
    print('Log Ingestor running on port 3000...')
    log_ingestor.serve_forever()

    
#Query-Interface

import json

logs = []  # Consider using a shared data structure or a database in a real-world scenario

def filter_logs(filters):
    return [log for log in logs if all(log.get(key) == filters[key] for key in filters)]

def main():
    while True:
        print("Enter a query:")
        print("Example: {'level': 'error', 'message': 'Failed to connect to DB'}")
        query_str = input("> ")
        
        if query_str.lower() == 'exit':
            break

        try:
            query = json.loads(query_str)
            results = filter_logs(query)
            print(json.dumps(results, indent=2))
        except json.JSONDecodeError as e:
            print(f"Error parsing query: {e}")

if __name__ == '__main__':
    main()
