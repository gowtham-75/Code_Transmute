# Code_Transmute
Transformation of code to another language


zip file
def extract_sql_files(zip_path, sql_dir_name="sql_files"):
    # Create the SQL files directory if it doesn't exist
    os.makedirs(sql_dir_name, exist_ok=True)
    sql_files_content = {}
    code_text = ""
    
    with zipfile.ZipFile(zip_path, 'r') as zip_ref:
        for file_name in zip_ref.namelist():
            # Check if the file is a .pks or .pkb file
            if file_name.endswith('.pks') or file_name.endswith('.pkb'):
                # Extract each file to the specified directory
                zip_ref.extract(file_name, sql_dir_name)
                
                # Read the content of the file to store it (optional)
                with zip_ref.open(file_name) as file:
                    sql_files_content[file_name] = file.read().decode('utf-8')
                
                with zip_ref.open(file_name) as file:
                    code_text += f"__________{file.name}__________\n"
                    code_text += file.read().decode('utf-8')
                    code_text += "\n\n"
    
    return sql_files_content, code_text
