# TextProcessor

A repository focused on processing and summarizing text from PDF files or URLs. The program can read content from local PDF files, PDF URLs, and HTML URLs, preprocess the text, extract Subject-Verb-Object (SVO) structures, extract keywords, create a keyword map, and provide a summary of the text.

## Features

- Reads text from local PDF files, PDF URLs, and HTML URLs.
- Preprocesses the text to remove stop words.
- Extracts Subject-Verb-Object (SVO) structures.
- Extracts keywords based on frequency.
- Creates a keyword map showing related SVOs.
- Provides a summary of the text.

## Installation

To install and set up this project, follow these steps:

1. Clone the repository:
    ```bash
    git clone https://github.com/git19112019/TextProcessor.git
    ```

2. Navigate to the project directory:
    ```bash
    cd TextProcessor
    ```

3. Ensure you have Python installed (preferably Python 3).

4. Install the required dependencies:
    ```bash
    pip install -r requirements.txt
    ```

## Usage

To use this project, follow these instructions:

1. Run the Python script `text_processor.py`:
    ```bash
    python text_processor.py <file_path_or_url>
    ```

2. Replace `<file_path_or_url>` with the path to the PDF file or URL you want to process.

3. The script will read the content, preprocess the text, extract keywords and SVOs, and display a summary and keyword map.

## Dependencies

This project requires the following dependencies:
- `spacy`
- `PyPDF2`
- `requests`
- `beautifulsoup4`
- `argparse`

You can install the dependencies using the following command:
```bash
pip install spacy PyPDF2 requests beautifulsoup4 argparse
```

## Framework

This project uses a compiler framework named `TextProcessorFramework` to integrate the idea, pseudo-code, and Python code. Below is the YAML configuration for the framework:

````yaml
compiler_framework:
  - idea: |
      Create a program that processes and summarizes text from PDF or URL. The program should be able to:
      - Read text from a local PDF file or a PDF/HTML URL.
      - Preprocess the text to remove stop words.
      - Extract Subject-Verb-Object (SVO) structures.
      - Extract keywords based on frequency.
      - Create a keyword map showing related SVOs.
      - Provide a summary of the text.
  - pseudo_code: |
      1. Import necessary libraries.
      2. Load the SpaCy language model.
      3. Define a function `read_text_from_pdf` to read text from a local PDF file or a PDF URL.
         - Check if the input is a URL or a local file path.
         - Use PyPDF2 to read the PDF content.
      4. Define a function `read_text_from_url` to read text from an HTML URL.
         - Use BeautifulSoup to parse the HTML content.
      5. Define a function `preprocess_text` to remove stop words from the text using SpaCy.
      6. Define a function `extract_svo` to extract Subject-Verb-Object structures from the text using SpaCy.
      7. Define a function `extract_keywords` to extract keywords based on frequency.
      8. Define a function `create_keyword_map` to create a keyword map showing related SVOs.
      9. Define a function `summarize_text` to preprocess the text, extract keywords and SVOs, and display the summary and keyword map.
      10. Define the main program to:
          - Parse command line arguments.
          - Check if the input is a URL or a local file path.
          - Read the text using the appropriate function.
          - Summarize the text if reading is successful.
  - python_code: |
      import spacy
      from collections import Counter
      import PyPDF2  # Để xử lý file PDF
      import requests  # Để tải file PDF từ URL
      from bs4 import BeautifulSoup  # Để xử lý nội dung HTML
      import io  # Để sử dụng BytesIO
      import argparse  # Để xử lý tham số dòng lệnh

      # Tải mô hình ngôn ngữ của SpaCy
      nlp = spacy.load("en_core_web_sm")

      # Hàm đọc nội dung từ file PDF (tải cục bộ hoặc từ URL)
      def read_text_from_pdf(file_path_or_url):
          try:
              # Kiểm tra nếu đường dẫn là URL
              if file_path_or_url.startswith("http"):
                  response = requests.get(file_path_or_url)
                  response.raise_for_status()  # Kiểm tra lỗi HTTP
                  file = io.BytesIO(response.content)  # Chuyển bytes thành file-like object
              else:
                  file = open(file_path_or_url, "rb")  # Mở file PDF cục bộ
              
              # Đọc nội dung file PDF
              pdf_reader = PyPDF2.PdfReader(file)
              text = ""
              for page in pdf_reader.pages:
                  text += page.extract_text()
              file.close()  # Đóng file nếu là file cục bộ
              return text.strip()
          except FileNotFoundError:
              print("File không tồn tại. Hãy kiểm tra lại đường dẫn.")
              return None
          except requests.exceptions.RequestException as e:
              print(f"Đã xảy ra lỗi khi tải URL: {e}")
              return None
          except Exception as e:
              print(f"Đã xảy ra lỗi: {e}")
              return None

      # Hàm tải và đọc nội dung từ URL HTML
      def read_text_from_url(url):
          try:
              response = requests.get(url)
              response.raise_for_status()  # Kiểm tra lỗi HTTP
              soup = BeautifulSoup(response.text, 'html.parser')
              # Trích xuất văn bản từ các thẻ <p>, <h1>, <h2>,...
              text = " ".join([p.get_text() for p in soup.find_all(['p', 'h1', 'h2', 'h3'])])
              return text.strip()
          except requests.exceptions.RequestException as e:
              print(f"Đã xảy ra lỗi khi tải URL: {e}")
              return None

      # Hàm tiền xử lý văn bản (loại bỏ stop words)
      def preprocess_text(text):
          doc = nlp(text)
          processed_text = " ".join([token.text for token in doc if token.is_alpha and not token.is_stop])
          return processed_text

      # Hàm trích xuất cấu trúc Subject-Verb-Object (SVO)
      def extract_svo(text):
          doc = nlp(text)
          svos = []
          for token in doc:
              if token.dep_ == "ROOT":  # Tìm động từ chính
                  subjects = [w.text for w in token.lefts if w.dep_ in ("nsubj", "nsubjpass")]
                  objects = [w.text for w in token.rights if w.dep_ in ("dobj", "pobj", "attr")]
                  if subjects and objects:
                      svos.append((subjects[0], token.text, objects[0]))
          return svos

      # Hàm trích xuất từ khóa dựa trên tần suất
      def extract_keywords(text, top_n=10):
          doc = nlp(text)
          keywords = [token.text for token in doc if token.is_alpha and not token.is_stop]
          keyword_freq = Counter(keywords)
          return keyword_freq.most_common(top_n)

      # Hàm hiển thị "map keyword" (bản đồ từ khóa)
      def create_keyword_map(keywords, svos):
          keyword_map = {}
          for keyword, _ in keywords:
              keyword_map[keyword] = {
                  "related_svos": [svo for svo in svos if keyword in svo]
              }
          
          print("\n=== Bản đồ từ khóa ===")
          for keyword, data in keyword_map.items():
              print(f"- Từ khóa: {keyword}")
              if data["related_svos"]:
                  for svo in data["related_svos"]:
                      print(f"  + SVO liên quan: {svo}")
              else:
                  print("  + Không có SVO liên quan.")

      # Hàm tóm tắt nội dung
      def summarize_text(text):
          print("\n=== Bắt đầu xử lý văn bản ===\n")
          processed_text = preprocess_text(text)
          svos = extract_svo(text)
          keywords = extract_keywords(processed_text)

          print("Từ khóa nổi bật:")
          for word, freq in keywords:
              print(f"- {word}: xuất hiện {freq} lần")

          print("\nCấu trúc SVO:")
          for subject, verb, obj in svos:
              print(f"- Chủ ngữ: {subject} → Động từ: {verb} → Tân ngữ: {obj}")

          print("\nTóm tắt:")
          key_elements = ", ".join([kw[0] for kw in keywords])
          print(f"Văn bản này nói về các yếu tố chính như: {key_elements}.")

          # Gọi hàm tạo bản đồ từ khóa
          create_keyword_map(keywords, svos)

      # Chương trình chính
      if __name__ == "__main__":
          parser = argparse.ArgumentParser(description="Process and summarize a text from PDF or URL.")
          parser.add_argument("file_path_or_url", help="Path to the PDF file or URL")
          args = parser.parse_args()

          file_path_or_url = args.file_path_or_url

          # Kiểm tra xem là URL HTML hay file PDF (URL hoặc cục bộ)
          if file_path_or_url.startswith("http") and file_path_or_url.endswith(".pdf"):
              text = read_text_from_pdf(file_path_or_url)  # Link PDF
          elif file_path_or_url.startswith("http"):
              text = read_text_from_url(file_path_or_url)  # Link HTML
          elif file_path_or_url.endswith(".pdf"):
              text = read_text_from_pdf(file_path_or_url)  # File PDF cục bộ
          else:
              print("Định dạng không được hỗ trợ.")
              text = None

          # Nếu đọc nội dung thành công, xử lý và tóm tắt văn bản
          if text:
              summarize_text(text)
