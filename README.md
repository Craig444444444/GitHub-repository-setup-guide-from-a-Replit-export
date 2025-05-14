import tarfile
import os
import ast
import hashlib
import sys
from datetime import datetime
from tqdm.auto import tqdm
from docx import Document
from docx.shared import Pt
from bandit.core import manager
from google.colab import files

class ChatbotAnalyzer:
    def __init__(self):
        self.structure = {}
        self.dependencies = set()
        self.chatbot_components = {}
        self.version_variants = {}
        self.creation_time = "2025-05-14 11:32:16"  # Updated to current time
        self.author = "Craig444444444"

    def process_tar(self, tar_path, extract_to):
        """Handles large chatbot TAR files with version merging"""
        if not os.path.exists(tar_path):
            print(f"Warning: TAR file not found: {tar_path}")
            return False
            
        print(f"Processing {os.path.basename(tar_path)}...")
        try:
            with tarfile.open(tar_path, "r:gz") as tar:
                members = []
                for member in tar:
                    if member.isfile():
                        members.append(member)
                
                for member in tqdm(members, desc="Extracting"):
                    try:
                        content = tar.extractfile(member)
                        if content is not None:
                            self._store_version(member.name, content.read())
                    except Exception as e:
                        print(f"Error extracting {member.name}: {str(e)[:50]}")

            self._merge_versions(extract_to)
            print(f"Successfully processed and merged files to: {extract_to}")
            return True
            
        except Exception as e:
            print(f"Error processing TAR file {tar_path}: {str(e)}")
            return False

    def _store_version(self, filename, content):
        """Track different versions of chatbot components"""
        try:
            file_hash = hashlib.md5(content).hexdigest()
            if filename not in self.version_variants:
                self.version_variants[filename] = {}
            self.version_variants[filename][file_hash] = content
        except Exception as e:
            print(f"Error storing version for {filename}: {str(e)}")

    def _merge_versions(self, extract_path):
        """Merge different versions with conflict resolution"""
        try:
            os.makedirs(extract_path, exist_ok=True)
            for filename, versions in self.version_variants.items():
                try:
                    # Create full path including directories
                    full_path = os.path.join(extract_path, filename)
                    os.makedirs(os.path.dirname(full_path), exist_ok=True)
                    
                    if len(versions) == 1:
                        # Single version - write directly
                        with open(full_path, 'wb') as f:
                            f.write(next(iter(versions.values())))
                    else:
                        # Multiple versions - create versioned files
                        base, ext = os.path.splitext(filename)
                        for i, (hash_val, content) in enumerate(versions.items(), 1):
                            version_path = f"{base}_v{i}{ext}"
                            full_version_path = os.path.join(extract_path, version_path)
                            with open(full_version_path, 'wb') as f:
                                f.write(content)
                                
                except Exception as e:
                    print(f"Error merging file {filename}: {str(e)}")
                    continue
                    
        except Exception as e:
            print(f"Error in merge process: {str(e)}")
            raise

def main():
    """Main execution function"""
    print(f"Starting analysis at {datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S')} UTC")
    print(f"User: {os.getenv('USER', 'Craig444444444')}")
    
    success = False
    try:
        analyzer = ChatbotAnalyzer()
        
        # Process TAR files
        tar_files = [
            '/content/ReplitExport-craighckby.tar.gz',
            '/content/ReplitExport-craighckby (1).tar.gz'
        ]
        
        for tar_file in tar_files:
            if analyzer.process_tar(tar_file, '/content/chatbot'):
                success = True
        
        if success:
            print("\nProcessing completed successfully!")
            if os.path.exists('/content/chatbot'):
                print(f"Files extracted to: {'/content/chatbot'}")
                # List some of the extracted files
                try:
                    files_list = os.listdir('/content/chatbot')
                    print("\nExtracted files (up to first 5):")
                    for f in files_list[:5]:
                        print(f"- {f}")
                except Exception as e:
                    print(f"Error listing files: {str(e)}")
            else:
                print("Warning: Output directory not found")
        else:
            print("\nNo TAR files were processed successfully")
            
    except Exception as e:
        print(f"\nFatal error: {str(e)}")
        return 1

    return 0

if __name__ == "__main__":
    main()
