# Module 9: Working with Excel and PDFs (2 Hours)

## Learning Objectives
- Master Excel file manipulation using `openpyxl`
- Process PDF documents with `PyPDF2` and `pdfplumber`
- Automate report generation workflows
- Extract and modify document data programmatically

---

## 9.1 Working with Excel Files

### Installing Required Libraries
```bash
pip install openpyxl pandas xlsxwriter
```

### Basic Excel Operations with openpyxl
```python
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, Alignment, PatternFill
from openpyxl.chart import BarChart, Reference
import logging

logger = logging.getLogger(__name__)

def create_basic_excel():
    """Create a basic Excel file with data"""
    # Create a new workbook
    wb = Workbook()
    ws = wb.active
    ws.title = "Sales Data"
    
    # Add headers
    headers = ["Product", "Q1", "Q2", "Q3", "Q4", "Total"]
    ws.append(headers)
    
    # Add data
    data = [
        ["Laptops", 100, 120, 110, 150],
        ["Phones", 200, 180, 220, 250],
        ["Tablets", 80, 90, 85, 95],
        ["Monitors", 60, 70, 65, 80]
    ]
    
    for row in data:
        # Calculate total
        total = sum(row[1:])
        row.append(total)
        ws.append(row)
    
    # Save the workbook
    wb.save("sales_report.xlsx")
    logger.info("Basic Excel file created successfully")

create_basic_excel()
```

### Reading Excel Files
```python
from openpyxl import load_workbook
import logging

logger = logging.getLogger(__name__)

def read_excel_data(filename):
    """Read data from Excel file"""
    try:
        wb = load_workbook(filename)
        ws = wb.active
        
        logger.info(f"Reading data from {filename}")
        logger.info(f"Sheet name: {ws.title}")
        logger.info(f"Dimensions: {ws.max_row} rows x {ws.max_column} columns")
        
        # Read all data
        data = []
        for row in ws.iter_rows(values_only=True):
            data.append(row)
        
        # Display headers and first few rows
        headers = data[0]
        logger.info(f"Headers: {headers}")
        
        for i, row in enumerate(data[1:4], 1):  # First 3 data rows
            logger.info(f"Row {i}: {row}")
        
        return data
        
    except FileNotFoundError:
        logger.error(f"File {filename} not found")
        return None
    except Exception as e:
        logger.error(f"Error reading Excel file: {e}")
        return None

# Usage
data = read_excel_data("sales_report.xlsx")
```

### Advanced Excel Formatting
```python
from openpyxl import Workbook
from openpyxl.styles import Font, Alignment, PatternFill, Border, Side
from openpyxl.chart import BarChart, Reference

def create_formatted_report():
    """Create a professionally formatted Excel report"""
    wb = Workbook()
    ws = wb.active
    ws.title = "Monthly Sales Report"
    
    # Define styles
    header_font = Font(bold=True, color="FFFFFF")
    header_fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
    border = Border(
        left=Side(style='thin'),
        right=Side(style='thin'),
        top=Side(style='thin'),
        bottom=Side(style='thin')
    )
    
    # Add title
    ws.merge_cells('A1:F1')
    ws['A1'] = "QUARTERLY SALES REPORT 2024"
    ws['A1'].font = Font(bold=True, size=16)
    ws['A1'].alignment = Alignment(horizontal='center')
    
    # Add headers
    headers = ["Product", "Q1", "Q2", "Q3", "Q4", "Total"]
    for col, header in enumerate(headers, 1):
        cell = ws.cell(row=3, column=col, value=header)
        cell.font = header_font
        cell.fill = header_fill
        cell.alignment = Alignment(horizontal='center')
        cell.border = border
    
    # Add data with formatting
    data = [
        ["Laptops", 100, 120, 110, 150],
        ["Phones", 200, 180, 220, 250],
        ["Tablets", 80, 90, 85, 95],
        ["Monitors", 60, 70, 65, 80]
    ]
    
    for row_idx, row_data in enumerate(data, 4):
        for col_idx, value in enumerate(row_data, 1):
            cell = ws.cell(row=row_idx, column=col_idx, value=value)
            cell.border = border
            if col_idx > 1:  # Numeric columns
                cell.alignment = Alignment(horizontal='right')
        
        # Add total formula
        total_cell = ws.cell(row=row_idx, column=6, value=f"=SUM(B{row_idx}:E{row_idx})")
        total_cell.font = Font(bold=True)
        total_cell.border = border
        total_cell.alignment = Alignment(horizontal='right')
    
    # Add chart
    chart = BarChart()
    chart.title = "Quarterly Sales by Product"
    
    # Data for chart
    data_ref = Reference(ws, min_col=2, min_row=3, max_col=5, max_row=7)
    categories = Reference(ws, min_col=1, min_row=4, max_row=7)
    
    chart.add_data(data_ref, titles_from_data=True)
    chart.set_categories(categories)
    
    # Add chart to worksheet
    ws.add_chart(chart, "H3")
    
    # Adjust column widths
    for column in ws.columns:
        max_length = 0
        column_letter = column[0].column_letter
        for cell in column:
            try:
                if len(str(cell.value)) > max_length:
                    max_length = len(str(cell.value))
            except:
                pass
        adjusted_width = min(max_length + 2, 20)
        ws.column_dimensions[column_letter].width = adjusted_width
    
    wb.save("formatted_sales_report.xlsx")
    logger.info("Formatted Excel report created successfully")

create_formatted_report()
```

### Data Analysis with Pandas and Excel
```python
import pandas as pd
import logging

logger = logging.getLogger(__name__)

def analyze_sales_data():
    """Analyze sales data using pandas"""
    # Create sample data
    data = {
        'Product': ['Laptops', 'Phones', 'Tablets', 'Monitors'] * 4,
        'Quarter': ['Q1', 'Q1', 'Q1', 'Q1', 'Q2', 'Q2', 'Q2', 'Q2',
                   'Q3', 'Q3', 'Q3', 'Q3', 'Q4', 'Q4', 'Q4', 'Q4'],
        'Sales': [100, 200, 80, 60, 120, 180, 90, 70,
                 110, 220, 85, 65, 150, 250, 95, 80],
        'Revenue': [50000, 80000, 24000, 18000, 60000, 72000, 27000, 21000,
                   55000, 88000, 25500, 19500, 75000, 100000, 28500, 24000]
    }
    
    df = pd.DataFrame(data)
    
    # Basic analysis
    logger.info("Sales Data Analysis")
    logger.info(f"Total records: {len(df)}")
    logger.info(f"Products: {df['Product'].unique()}")
    
    # Summary statistics
    summary = df.groupby('Product').agg({
        'Sales': ['sum', 'mean'],
        'Revenue': ['sum', 'mean']
    }).round(2)
    
    logger.info("Summary by Product:")
    logger.info(summary.to_string())
    
    # Create Excel file with multiple sheets
    with pd.ExcelWriter('sales_analysis.xlsx', engine='openpyxl') as writer:
        # Raw data sheet
        df.to_excel(writer, sheet_name='Raw Data', index=False)
        
        # Summary sheet
        summary.to_excel(writer, sheet_name='Summary')
        
        # Pivot table
        pivot = df.pivot_table(
            values='Sales',
            index='Product',
            columns='Quarter',
            aggfunc='sum',
            fill_value=0
        )
        pivot.to_excel(writer, sheet_name='Pivot Table')
    
    logger.info("Sales analysis Excel file created successfully")

analyze_sales_data()
```

---

## 9.2 Working with PDF Files

### Installing PDF Libraries
```bash
pip install PyPDF2 pdfplumber reportlab
```

### Reading PDF Content
```python
import PyPDF2
import pdfplumber
import logging

logger = logging.getLogger(__name__)

def read_pdf_pypdf2(filename):
    """Read PDF using PyPDF2"""
    try:
        with open(filename, 'rb') as file:
            pdf_reader = PyPDF2.PdfReader(file)
            
            logger.info(f"PDF Info for {filename}:")
            logger.info(f"Number of pages: {len(pdf_reader.pages)}")
            
            # Extract text from all pages
            full_text = ""
            for page_num, page in enumerate(pdf_reader.pages):
                text = page.extract_text()
                full_text += text
                logger.info(f"Page {page_num + 1} - Characters: {len(text)}")
            
            return full_text
            
    except FileNotFoundError:
        logger.error(f"PDF file {filename} not found")
        return None
    except Exception as e:
        logger.error(f"Error reading PDF: {e}")
        return None

def read_pdf_pdfplumber(filename):
    """Read PDF using pdfplumber (better for tables and structured data)"""
    try:
        with pdfplumber.open(filename) as pdf:
            logger.info(f"PDF Info for {filename}:")
            logger.info(f"Number of pages: {len(pdf.pages)}")
            
            full_text = ""
            tables = []
            
            for page_num, page in enumerate(pdf.pages):
                # Extract text
                text = page.extract_text()
                if text:
                    full_text += text + "\n"
                
                # Extract tables
                page_tables = page.extract_tables()
                if page_tables:
                    tables.extend(page_tables)
                    logger.info(f"Page {page_num + 1} - Found {len(page_tables)} tables")
            
            return {"text": full_text, "tables": tables}
            
    except FileNotFoundError:
        logger.error(f"PDF file {filename} not found")
        return None
    except Exception as e:
        logger.error(f"Error reading PDF: {e}")
        return None
```

### Creating PDFs with ReportLab
```python
from reportlab.lib.pagesizes import letter, A4
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch
from reportlab.lib import colors
import logging

logger = logging.getLogger(__name__)

def create_sales_report_pdf():
    """Create a professional PDF report"""
    doc = SimpleDocTemplate("sales_report.pdf", pagesize=A4)
    styles = getSampleStyleSheet()
    story = []
    
    # Title
    title_style = ParagraphStyle(
        'CustomTitle',
        parent=styles['Heading1'],
        fontSize=24,
        spaceAfter=30,
        alignment=1  # Center alignment
    )
    title = Paragraph("QUARTERLY SALES REPORT 2024", title_style)
    story.append(title)
    
    # Introduction
    intro_text = """
    This report provides a comprehensive overview of our sales performance 
    across all product categories for the fiscal year 2024. The data includes 
    quarterly breakdowns and year-over-year comparisons.
    """
    intro = Paragraph(intro_text, styles['Normal'])
    story.append(intro)
    story.append(Spacer(1, 12))
    
    # Sales data table
    data = [
        ['Product', 'Q1', 'Q2', 'Q3', 'Q4', 'Total'],
        ['Laptops', '100', '120', '110', '150', '480'],
        ['Phones', '200', '180', '220', '250', '850'],
        ['Tablets', '80', '90', '85', '95', '350'],
        ['Monitors', '60', '70', '65', '80', '275'],
        ['TOTAL', '440', '460', '480', '575', '1,955']
    ]
    
    table = Table(data)
    table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('FONTSIZE', (0, 0), (-1, 0), 14),
        ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
        ('BACKGROUND', (0, 1), (-1, -2), colors.beige),
        ('BACKGROUND', (0, -1), (-1, -1), colors.lightgrey),
        ('FONTNAME', (0, -1), (-1, -1), 'Helvetica-Bold'),
        ('GRID', (0, 0), (-1, -1), 1, colors.black)
    ]))
    
    story.append(table)
    story.append(Spacer(1, 12))
    
    # Summary section
    summary_title = Paragraph("Executive Summary", styles['Heading2'])
    story.append(summary_title)
    
    summary_text = """
    • Total units sold: 1,955 units
    • Best performing quarter: Q4 with 575 units
    • Top product: Phones with 850 units sold
    • Growth trend: Steady increase throughout the year
    """
    summary = Paragraph(summary_text, styles['Normal'])
    story.append(summary)
    
    # Build PDF
    doc.build(story)
    logger.info("PDF report created successfully")

create_sales_report_pdf()
```

### PDF Processing Automation
```python
import PyPDF2
import pdfplumber
import pandas as pd
import os
import logging

logger = logging.getLogger(__name__)

class PDFProcessor:
    def __init__(self, input_folder="pdfs", output_folder="processed"):
        self.input_folder = input_folder
        self.output_folder = output_folder
        
        # Create folders if they don't exist
        os.makedirs(input_folder, exist_ok=True)
        os.makedirs(output_folder, exist_ok=True)
    
    def extract_text_from_pdfs(self):
        """Extract text from all PDFs in input folder"""
        results = []
        
        for filename in os.listdir(self.input_folder):
            if filename.lower().endswith('.pdf'):
                filepath = os.path.join(self.input_folder, filename)
                
                try:
                    with pdfplumber.open(filepath) as pdf:
                        text = ""
                        for page in pdf.pages:
                            page_text = page.extract_text()
                            if page_text:
                                text += page_text + "\n"
                        
                        results.append({
                            'filename': filename,
                            'pages': len(pdf.pages),
                            'text_length': len(text),
                            'text': text[:500] + "..." if len(text) > 500 else text
                        })
                        
                        logger.info(f"Processed {filename}: {len(pdf.pages)} pages")
                        
                except Exception as e:
                    logger.error(f"Error processing {filename}: {e}")
        
        return results
    
    def extract_tables_from_pdfs(self):
        """Extract tables from all PDFs"""
        all_tables = []
        
        for filename in os.listdir(self.input_folder):
            if filename.lower().endswith('.pdf'):
                filepath = os.path.join(self.input_folder, filename)
                
                try:
                    with pdfplumber.open(filepath) as pdf:
                        for page_num, page in enumerate(pdf.pages):
                            tables = page.extract_tables()
                            
                            for table_num, table in enumerate(tables):
                                if table:  # Check if table is not empty
                                    df = pd.DataFrame(table[1:], columns=table[0])
                                    
                                    # Save table as CSV
                                    output_filename = f"{filename}_{page_num+1}_{table_num+1}.csv"
                                    output_path = os.path.join(self.output_folder, output_filename)
                                    df.to_csv(output_path, index=False)
                                    
                                    all_tables.append({
                                        'source_file': filename,
                                        'page': page_num + 1,
                                        'table_number': table_num + 1,
                                        'rows': len(df),
                                        'columns': len(df.columns),
                                        'output_file': output_filename
                                    })
                                    
                                    logger.info(f"Extracted table from {filename}, page {page_num+1}")
                        
                except Exception as e:
                    logger.error(f"Error extracting tables from {filename}: {e}")
        
        return all_tables
    
    def merge_pdfs(self, output_filename="merged.pdf"):
        """Merge all PDFs in input folder"""
        pdf_writer = PyPDF2.PdfWriter()
        
        for filename in sorted(os.listdir(self.input_folder)):
            if filename.lower().endswith('.pdf'):
                filepath = os.path.join(self.input_folder, filename)
                
                try:
                    with open(filepath, 'rb') as file:
                        pdf_reader = PyPDF2.PdfReader(file)
                        
                        for page in pdf_reader.pages:
                            pdf_writer.add_page(page)
                        
                        logger.info(f"Added {filename} to merged PDF")
                        
                except Exception as e:
                    logger.error(f"Error adding {filename} to merge: {e}")
        
        # Save merged PDF
        output_path = os.path.join(self.output_folder, output_filename)
        with open(output_path, 'wb') as output_file:
            pdf_writer.write(output_file)
        
        logger.info(f"Merged PDF saved as {output_path}")

# Usage example
def demonstrate_pdf_processing():
    processor = PDFProcessor()
    
    # Extract text from all PDFs
    text_results = processor.extract_text_from_pdfs()
    
    # Save text extraction results
    if text_results:
        df = pd.DataFrame(text_results)
        df.to_csv('processed/text_extraction_results.csv', index=False)
        logger.info("Text extraction results saved")
    
    # Extract tables from all PDFs
    table_results = processor.extract_tables_from_pdfs()
    
    # Save table extraction results
    if table_results:
        df = pd.DataFrame(table_results)
        df.to_csv('processed/table_extraction_results.csv', index=False)
        logger.info("Table extraction results saved")
    
    # Merge all PDFs
    processor.merge_pdfs()

# Run the demonstration
demonstrate_pdf_processing()
```

---

## 9.3 Practical Automation Examples

### Automated Report Generation
```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.chart import BarChart, Reference
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle
from reportlab.lib import colors
import logging

logger = logging.getLogger(__name__)

class ReportGenerator:
    def __init__(self, data_source):
        self.data = pd.read_csv(data_source) if isinstance(data_source, str) else data_source
        logger.info(f"Data loaded: {len(self.data)} rows")
    
    def generate_excel_report(self, filename="automated_report.xlsx"):
        """Generate comprehensive Excel report"""
        with pd.ExcelWriter(filename, engine='openpyxl') as writer:
            # Summary sheet
            summary = self.data.groupby('Product').agg({
                'Sales': ['sum', 'mean', 'std'],
                'Revenue': ['sum', 'mean']
            }).round(2)
            summary.to_excel(writer, sheet_name='Summary')
            
            # Monthly trends
            if 'Month' in self.data.columns:
                monthly = self.data.groupby('Month')['Sales'].sum()
                monthly.to_excel(writer, sheet_name='Monthly Trends')
            
            # Raw data
            self.data.to_excel(writer, sheet_name='Raw Data', index=False)
            
            logger.info(f"Excel report generated: {filename}")
    
    def generate_pdf_report(self, filename="automated_report.pdf"):
        """Generate PDF report"""
        doc = SimpleDocTemplate(filename)
        elements = []
        
        # Convert data to table format
        table_data = [list(self.data.columns)]
        for _, row in self.data.head(10).iterrows():  # First 10 rows
            table_data.append(list(row))
        
        table = Table(table_data)
        table.setStyle(TableStyle([
            ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
            ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
            ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
            ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
            ('FONTSIZE', (0, 0), (-1, 0), 12),
            ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
            ('GRID', (0, 0), (-1, -1), 1, colors.black)
        ]))
        
        elements.append(table)
        doc.build(elements)
        logger.info(f"PDF report generated: {filename}")

# Example usage
sample_data = pd.DataFrame({
    'Product': ['Laptops', 'Phones', 'Tablets'] * 4,
    'Month': ['Jan', 'Jan', 'Jan', 'Feb', 'Feb', 'Feb', 
              'Mar', 'Mar', 'Mar', 'Apr', 'Apr', 'Apr'],
    'Sales': [100, 200, 80, 120, 180, 90, 110, 220, 85, 150, 250, 95],
    'Revenue': [50000, 80000, 24000, 60000, 72000, 27000, 
                55000, 88000, 25500, 75000, 100000, 28500]
})

generator = ReportGenerator(sample_data)
generator.generate_excel_report()
generator.generate_pdf_report()
```

### Document Processing Pipeline
```python
import os
import pandas as pd
import pdfplumber
from openpyxl import load_workbook
import logging

logger = logging.getLogger(__name__)

class DocumentProcessor:
    def __init__(self, input_dir="documents", output_dir="processed_documents"):
        self.input_dir = input_dir
        self.output_dir = output_dir
        
        os.makedirs(input_dir, exist_ok=True)
        os.makedirs(output_dir, exist_ok=True)
    
    def process_all_documents(self):
        """Process all Excel and PDF files in input directory"""
        results = {
            'excel_files': [],
            'pdf_files': [],
            'errors': []
        }
        
        for filename in os.listdir(self.input_dir):
            filepath = os.path.join(self.input_dir, filename)
            
            try:
                if filename.lower().endswith('.xlsx'):
                    result = self.process_excel_file(filepath)
                    results['excel_files'].append(result)
                
                elif filename.lower().endswith('.pdf'):
                    result = self.process_pdf_file(filepath)
                    results['pdf_files'].append(result)
                
            except Exception as e:
                error_info = {'filename': filename, 'error': str(e)}
                results['errors'].append(error_info)
                logger.error(f"Error processing {filename}: {e}")
        
        self.generate_processing_report(results)
        return results
    
    def process_excel_file(self, filepath):
        """Extract data and metadata from Excel file"""
        filename = os.path.basename(filepath)
        wb = load_workbook(filepath)
        
        result = {
            'filename': filename,
            'type': 'Excel',
            'sheets': [],
            'total_rows': 0,
            'total_columns': 0
        }
        
        for sheet_name in wb.sheetnames:
            ws = wb[sheet_name]
            sheet_info = {
                'name': sheet_name,
                'rows': ws.max_row,
                'columns': ws.max_column,
                'has_data': ws.max_row > 1
            }
            
            result['sheets'].append(sheet_info)
            result['total_rows'] += ws.max_row
            result['total_columns'] = max(result['total_columns'], ws.max_column)
            
            # Extract data if it has content
            if sheet_info['has_data']:
                data = []
                for row in ws.iter_rows(values_only=True):
                    data.append(row)
                
                # Save as CSV
                df = pd.DataFrame(data[1:], columns=data[0])
                csv_filename = f"{filename}_{sheet_name}.csv"
                csv_path = os.path.join(self.output_dir, csv_filename)
                df.to_csv(csv_path, index=False)
                
                sheet_info['csv_output'] = csv_filename
        
        logger.info(f"Processed Excel file: {filename}")
        return result
    
    def process_pdf_file(self, filepath):
        """Extract text and tables from PDF file"""
        filename = os.path.basename(filepath)
        
        result = {
            'filename': filename,
            'type': 'PDF',
            'pages': 0,
            'text_length': 0,
            'tables_found': 0,
            'outputs': []
        }
        
        with pdfplumber.open(filepath) as pdf:
            result['pages'] = len(pdf.pages)
            full_text = ""
            
            for page_num, page in enumerate(pdf.pages):
                # Extract text
                page_text = page.extract_text()
                if page_text:
                    full_text += page_text + "\n"
                
                # Extract tables
                tables = page.extract_tables()
                for table_num, table in enumerate(tables):
                    if table and len(table) > 1:
                        df = pd.DataFrame(table[1:], columns=table[0])
                        table_filename = f"{filename}_page{page_num+1}_table{table_num+1}.csv"
                        table_path = os.path.join(self.output_dir, table_filename)
                        df.to_csv(table_path, index=False)
                        
                        result['outputs'].append(table_filename)
                        result['tables_found'] += 1
            
            # Save extracted text
            if full_text.strip():
                text_filename = f"{filename}_text.txt"
                text_path = os.path.join(self.output_dir, text_filename)
                with open(text_path, 'w', encoding='utf-8') as f:
                    f.write(full_text)
                
                result['text_length'] = len(full_text)
                result['outputs'].append(text_filename)
        
        logger.info(f"Processed PDF file: {filename}")
        return result
    
    def generate_processing_report(self, results):
        """Generate a summary report of processing results"""
        report_data = []
        
        # Process Excel results
        for excel_result in results['excel_files']:
            report_data.append({
                'Filename': excel_result['filename'],
                'Type': excel_result['type'],
                'Sheets/Pages': len(excel_result['sheets']),
                'Total Rows': excel_result['total_rows'],
                'Outputs': len([s for s in excel_result['sheets'] if 'csv_output' in s]),
                'Status': 'Success'
            })
        
        # Process PDF results
        for pdf_result in results['pdf_files']:
            report_data.append({
                'Filename': pdf_result['filename'],
                'Type': pdf_result['type'],
                'Sheets/Pages': pdf_result['pages'],
                'Total Rows': 'N/A',
                'Outputs': len(pdf_result['outputs']),
                'Status': 'Success'
            })
        
        # Process errors
        for error in results['errors']:
            report_data.append({
                'Filename': error['filename'],
                'Type': 'Unknown',
                'Sheets/Pages': 'N/A',
                'Total Rows': 'N/A',
                'Outputs': 0,
                'Status': f"Error: {error['error']}"
            })
        
        # Save report
        df = pd.DataFrame(report_data)
        report_path = os.path.join(self.output_dir, 'processing_report.xlsx')
        df.to_excel(report_path, index=False)
        
        logger.info(f"Processing report saved: {report_path}")
        logger.info(f"Total files processed: {len(report_data)}")
        logger.info(f"Successful: {len([r for r in report_data if r['Status'] == 'Success'])}")
        logger.info(f"Errors: {len(results['errors'])}")

# Usage example
processor = DocumentProcessor()
results = processor.process_all_documents()
```

---

## 9.4 Best Practices and Error Handling

### Robust File Processing
```python
import os
import pandas as pd
from openpyxl import load_workbook
import pdfplumber
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

class RobustDocumentHandler:
    def __init__(self):
        self.supported_extensions = {
            'excel': ['.xlsx', '.xls'],
            'pdf': ['.pdf'],
            'csv': ['.csv']
        }
    
    def validate_file(self, filepath):
        """Validate file exists and is supported"""
        file_path = Path(filepath)
        
        if not file_path.exists():
            raise FileNotFoundError(f"File not found: {filepath}")
        
        if not file_path.is_file():
            raise ValueError(f"Path is not a file: {filepath}")
        
        extension = file_path.suffix.lower()
        file_type = None
        
        for type_name, extensions in self.supported_extensions.items():
            if extension in extensions:
                file_type = type_name
                break
        
        if not file_type:
            raise ValueError(f"Unsupported file type: {extension}")
        
        return file_type
    
    def safe_excel_read(self, filepath, sheet_name=None):
        """Safely read Excel file with comprehensive error handling"""
        try:
            file_type = self.validate_file(filepath)
            if file_type != 'excel':
                raise ValueError(f"Expected Excel file, got {file_type}")
            
            logger.info(f"Reading Excel file: {filepath}")
            
            # Try to read with pandas first (faster for data)
            try:
                if sheet_name:
                    df = pd.read_excel(filepath, sheet_name=sheet_name)
                else:
                    df = pd.read_excel(filepath)
                
                logger.info(f"Successfully read {len(df)} rows from {filepath}")
                return df
                
            except Exception as pandas_error:
                logger.warning(f"Pandas read failed, trying openpyxl: {pandas_error}")
                
                # Fallback to openpyxl for more complex files
                wb = load_workbook(filepath)
                ws = wb.active if not sheet_name else wb[sheet_name]
                
                data = []
                for row in ws.iter_rows(values_only=True):
                    data.append(row)
                
                if data:
                    df = pd.DataFrame(data[1:], columns=data[0])
                    logger.info(f"Successfully read {len(df)} rows using openpyxl")
                    return df
                else:
                    logger.warning("No data found in Excel file")
                    return pd.DataFrame()
        
        except Exception as e:
            logger.error(f"Error reading Excel file {filepath}: {e}")
            raise
    
    def safe_pdf_read(self, filepath):
        """Safely read PDF file with error handling"""
        try:
            file_type = self.validate_file(filepath)
            if file_type != 'pdf':
                raise ValueError(f"Expected PDF file, got {file_type}")
            
            logger.info(f"Reading PDF file: {filepath}")
            
            result = {
                'text': '',
                'tables': [],
                'metadata': {}
            }
            
            with pdfplumber.open(filepath) as pdf:
                result['metadata'] = {
                    'pages': len(pdf.pages),
                    'title': getattr(pdf.metadata, 'title', 'Unknown'),
                    'author': getattr(pdf.metadata, 'author', 'Unknown')
                }
                
                for page_num, page in enumerate(pdf.pages):
                    try:
                        # Extract text
                        page_text = page.extract_text()
                        if page_text:
                            result['text'] += f"--- Page {page_num + 1} ---\n{page_text}\n"
                        
                        # Extract tables
                        tables = page.extract_tables()
                        for table in tables:
                            if table and len(table) > 1:
                                result['tables'].append({
                                    'page': page_num + 1,
                                    'data': table
                                })
                    
                    except Exception as page_error:
                        logger.warning(f"Error processing page {page_num + 1}: {page_error}")
                        continue
            
            logger.info(f"Successfully read PDF: {result['metadata']['pages']} pages, "
                       f"{len(result['text'])} characters, {len(result['tables'])} tables")
            
            return result
        
        except Exception as e:
            logger.error(f"Error reading PDF file {filepath}: {e}")
            raise
    
    def batch_process_directory(self, directory_path, output_dir="processed"):
        """Process all supported files in a directory"""
        directory = Path(directory_path)
        output_directory = Path(output_dir)
        
        if not directory.exists():
            raise FileNotFoundError(f"Directory not found: {directory_path}")
        
        output_directory.mkdir(exist_ok=True)
        
        results = {
            'successful': [],
            'failed': [],
            'summary': {}
        }
        
        # Get all supported files
        all_files = []
        for type_name, extensions in self.supported_extensions.items():
            for ext in extensions:
                all_files.extend(directory.glob(f"*{ext}"))
        
        logger.info(f"Found {len(all_files)} supported files to process")
        
        for file_path in all_files:
            try:
                file_type = self.validate_file(file_path)
                
                if file_type == 'excel':
                    df = self.safe_excel_read(file_path)
                    output_path = output_directory / f"{file_path.stem}_data.csv"
                    df.to_csv(output_path, index=False)
                    
                    results['successful'].append({
                        'file': str(file_path),
                        'type': file_type,
                        'output': str(output_path),
                        'rows': len(df)
                    })
                
                elif file_type == 'pdf':
                    pdf_data = self.safe_pdf_read(file_path)
                    
                    # Save text
                    text_output = output_directory / f"{file_path.stem}_text.txt"
                    with open(text_output, 'w', encoding='utf-8') as f:
                        f.write(pdf_data['text'])
                    
                    # Save tables
                    table_outputs = []
                    for i, table_info in enumerate(pdf_data['tables']):
                        table_df = pd.DataFrame(table_info['data'][1:], 
                                              columns=table_info['data'][0])
                        table_output = output_directory / f"{file_path.stem}_table_{i+1}.csv"
                        table_df.to_csv(table_output, index=False)
                        table_outputs.append(str(table_output))
                    
                    results['successful'].append({
                        'file': str(file_path),
                        'type': file_type,
                        'text_output': str(text_output),
                        'table_outputs': table_outputs,
                        'pages': pdf_data['metadata']['pages']
                    })
            
            except Exception as e:
                results['failed'].append({
                    'file': str(file_path),
                    'error': str(e)
                })
                logger.error(f"Failed to process {file_path}: {e}")
        
        # Generate summary
        results['summary'] = {
            'total_files': len(all_files),
            'successful': len(results['successful']),
            'failed': len(results['failed']),
            'success_rate': len(results['successful']) / len(all_files) * 100 if all_files else 0
        }
        
        # Save processing report
        report_data = []
        for item in results['successful']:
            report_data.append({
                'File': Path(item['file']).name,
                'Type': item['type'],
                'Status': 'Success',
                'Details': f"Processed successfully - {item.get('rows', 'N/A')} rows"
            })
        
        for item in results['failed']:
            report_data.append({
                'File': Path(item['file']).name,
                'Type': 'Unknown',
                'Status': 'Failed',
                'Details': item['error']
            })
        
        report_df = pd.DataFrame(report_data)
        report_path = output_directory / 'processing_report.xlsx'
        report_df.to_excel(report_path, index=False)
        
        logger.info(f"Batch processing complete:")
        logger.info(f"  Total files: {results['summary']['total_files']}")
        logger.info(f"  Successful: {results['summary']['successful']}")
        logger.info(f"  Failed: {results['summary']['failed']}")
        logger.info(f"  Success rate: {results['summary']['success_rate']:.1f}%")
        logger.info(f"  Report saved: {report_path}")
        
        return results

# Example usage
handler = RobustDocumentHandler()

# Process individual files
try:
    excel_data = handler.safe_excel_read("sample.xlsx")
    pdf_data = handler.safe_pdf_read("sample.pdf")
except Exception as e:
    logger.error(f"Individual file processing error: {e}")

# Batch process directory
try:
    results = handler.batch_process_directory("documents")
except Exception as e:
    logger.error(f"Batch processing error: {e}")
```

---

## Summary and Best Practices

### Module 8: Logging and Debugging
**Key Takeaways:**
1. **Always use logging instead of print statements** for production code
2. **Configure appropriate log levels** based on your needs
3. **Use structured logging** with consistent formats
4. **Log to files** for persistent monitoring
5. **Include context** in your log messages
6. **Handle exceptions properly** with logging

### Module 9: Excel and PDF Processing
**Key Takeaways:**
1. **Use appropriate libraries** - `openpyxl` for Excel, `pdfplumber` for PDFs
2. **Always validate inputs** before processing
3. **Handle errors gracefully** with try-catch blocks
4. **Process data in chunks** for large files
5. **Save intermediate results** during batch processing
6. **Generate reports** for processing activities

### Common Pitfalls to Avoid
1. **Not handling file encoding issues**
2. **Forgetting to close file handles**
3. **Not validating file existence and permissions**
4. **Ignoring memory usage with large files**
5. **Not logging processing progress**
6. **Hardcoding file paths and configurations**

### Performance Tips
1. **Use pandas for large datasets**
2. **Process files in batches**
3. **Use generators for memory efficiency**
4. **Cache frequently accessed data**
5. **Optimize regex patterns**
6. **Use appropriate data types**

This completes the comprehensive training material for Modules 8 and 9, covering logging, debugging, Excel processing, and PDF manipulation with practical examples and best practices.
