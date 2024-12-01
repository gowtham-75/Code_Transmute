# Code_Transmute
Transformation of code to another language

import streamlit as st
st.set_page_config(
    page_title="Code convertor",
    page_icon="📜",
    layout="wide",
    initial_sidebar_state="expanded",
)

import os
import zipfile
import streamlit as st
from dotenv import load_dotenv
from langchain_core.prompts import PromptTemplate
from llm_model import llm_l, llm_Normal
from prompt_templates import (
    code_explain_prompt,
    java_code_gen_prompt,
    oo_design_prompt,
    mermaid_code,
    Bl_logic,
    Fun_doc,
    duplicate_redundant,
    Techenical,
    Func_overview,
    end
)

from mermaid import (generateDiagram,
                     generate_mermaid_process_flow_chart,
                     generateDiagram1,
                     generate_mermaid_process_flow_chart_TD,
                     generate_mermaid_process_flow_chart_TD_graph,
                     generate_mermaid_process_flow_chart_graph,
                     generate_mermaid_component
                     )
from pdf import pdf_convertion, pdf_convertion_with_mermaid
import streamlit.components.v1 as components
from mermaid_prompt import mermaid_code_generate,sequence_diagram,Mindmap,ER_Diagram,State_Diagram,class_diagram,Flowchart



load_dotenv()

# Folder to store extracted files
PKB_FOLDER = "./temp_files_all"

# Cache function to clear folder
def clear_folder(folder_path):
    try:
        if os.path.exists(folder_path):
            for file in os.listdir(folder_path):
                file_path = os.path.join(folder_path, file)
                if os.path.isfile(file_path):
                    os.remove(file_path)
                elif os.path.isdir(file_path):
                    for root, dirs, files in os.walk(file_path, topdown=False):
                        for name in files:
                            os.remove(os.path.join(root, name))
                        for name in dirs:
                            os.rmdir(os.path.join(root, name))
                    os.rmdir(file_path)
        else:
            os.makedirs(folder_path)
    except Exception as e:
        error=None
        # st.error(f"Error clearing folder {folder_path}: {e}")

# Cache function for extracting ZIP files
def extract_zip(zip_file_path, output_folder):
    try:
        if not os.path.exists(output_folder):
            os.makedirs(output_folder)

        with zipfile.ZipFile(zip_file_path, 'r') as zip_ref:
            zip_ref.extractall(output_folder)
        return zip_ref.namelist()
    except zipfile.BadZipFile:
        st.error("The uploaded file is not a valid ZIP file.")
        return []
    except Exception as e:
        st.error(f"An error occurred while extracting ZIP file: {e}")
        return []

# LLM call with prompt template
def execute(exec_prompt, code):
    """
    Execute LLM with provided prompt template
    """
    model = "gpt-4o"
    final_prompt = PromptTemplate.from_template(exec_prompt)
    formatted_prompt = final_prompt.format(PLSQL_CODE=code)
    llm_instance = llm_l(formatted_prompt, model)  # Create the LLM instance
    return llm_instance


# Check the proper diagram format
def mermaid_diagram(response):
    html_string=""
    if 'flowchart LR' in response:
        code_diagram =generate_mermaid_process_flow_chart(response)
        for item in code_diagram:                     
            data_diagram = f""" flowchart LR \n {item}"""                        
            html_string = generateDiagram1(data_diagram)
        return html_string,data_diagram
            
    elif 'flowchart TD' in response:
        code_diagram = generate_mermaid_process_flow_chart_TD(response)
        for item in code_diagram:                     
            data_diagram = f""" flowchart TD \n {item}"""                        
            html_string = generateDiagram1(data_diagram)
        return html_string,data_diagram
    
    elif 'graph LR' in response:
        code_diagram =generate_mermaid_process_flow_chart_graph(response)
        for item in code_diagram:                     
            data_diagram = f""" graph LR \n {item}"""                        
            html_string = generateDiagram1(data_diagram)
        return html_string,data_diagram
    
    elif 'graph TD' in response:
        code_diagram = generate_mermaid_process_flow_chart_TD_graph(response)
        for item in code_diagram:                     
            data_diagram = f""" graph TD \n {item}"""                        
            html_string = generateDiagram1(data_diagram)
        return html_string,data_diagram
    
    elif 'componentDiagram' in response:
        code_diagram = generate_mermaid_component(response)
        for item in code_diagram:                     
            data_diagram = f""" componentDiagram \n {item}"""                        
            html_string = generateDiagram1(data_diagram)
        return html_string,data_diagram
    
    return("_")
    

# Function to store results in session state or cache
def func(selected_option, code_text):
    # Create a unique key based on the selected option and code text
    cache_key = f"{selected_option}_{hash(code_text)}"
    
    # Check if the result is already cached in session state
    if cache_key in st.session_state:
        response = st.session_state[cache_key]
        st.write("Using cached response.")
    else:
        if selected_option == "Functional Document":
            if "doc0" not in st.session_state:
                response = execute(Fun_doc, code_text)
                st.session_state["doc0"] = response
            else:
                response = st.session_state["doc0"]
        
        elif selected_option == "Technical Document":
            if "doc5" not in st.session_state:
                response = execute(Techenical, code_text)
                st.session_state["doc5"] = response
            else:
                response = st.session_state["doc5"]
        
        elif selected_option == "Function Document overview":
            if "doc6" not in st.session_state:
                response = execute(Func_overview, code_text)
                st.session_state["doc6"] = response
            else:
                response = st.session_state["doc6"]
            
            # Generate the mermaid diagram and store it in session state
            st.session_state.dig1, st.session_state.dig_data = mermaid_diagram(response)
            # pdf_convertion_with_mermaid(response, st.session_state.dig_data)
            st.write(response)
            components.html(st.session_state.dig1, height=400, scrolling=True)
            return response

        else:
            st.error("Invalid option selected.")
            return None
        
        # Cache the response in session state
        st.session_state[cache_key] = response

    # Display the response
    st.write(response)
    return response



# Function to process `.pkb` files
def process_pkb_files(folder_path, option):
    responses = {}
    for root, dirs, files in os.walk(folder_path):  # Recursively walk through all folders
        for file_name in files:
            if file_name.endswith(".pkb"):
                file_path = os.path.join(root, file_name)
                try:
                    with open(file_path, "r") as file:
                        file_content = file.read()

                    # Pass the prompt to the LLM
                    
                    response = func(option, file_content)
                    responses[file_name] = response
                    st.write("**Procedure Name:**",file_name)
                    st.write(response)
                except Exception as e:
                    st.error(f"Error processing {file_name}: {e}")
    return responses

# Streamlit app
def main():
    # Initialize session state
    if "extracted_files" not in st.session_state:
        st.session_state["extracted_files"] = []
    if "selected_option" not in st.session_state:
        st.session_state["selected_option"] = None
    if "enable_doc_explain" not in st.session_state:
        st.session_state["enable_doc_explain"] = False

    st.title("PLSQL Code Processor")

    # Create tabs
    tab1, tab2, tab3 = st.tabs(["Extract Files", "View Files", "Generate Documentation"])

    # Tab 1: Extract Files
    with tab1:
        st.header("Extract Files")
        st.write("Upload a ZIP file containing `.pkb` files to extract them.")
        uploaded_file = st.file_uploader("Upload a ZIP file", type="zip")

        if st.button("Extract Files", key="extract_files"):
            clear_folder(PKB_FOLDER)  # Clear old data
            if uploaded_file:
                # Save the uploaded file to a temporary location
                zip_path = os.path.join(PKB_FOLDER, "uploaded.zip")
                with open(zip_path, "wb") as temp_file:
                    temp_file.write(uploaded_file.getbuffer())

                # Extract files
                st.session_state["extracted_files"] = extract_zip(zip_path, PKB_FOLDER)

                if st.session_state["extracted_files"]:
                    st.success(f"Extracted {len(st.session_state['extracted_files'])} files successfully!")
                    st.info(f"Extracted files: {', '.join(st.session_state['extracted_files'])}")
                else:
                    st.warning("No files were extracted from the ZIP file.")
            else:
                st.warning("Please upload a ZIP file first.")

    # Tab 2: View Files
    with tab2:
        st.header("View Extracted Files and Input Data")
        pkb_files = st.session_state.get("extracted_files", [])
        if pkb_files:
            st.success(f"Found {len(pkb_files)} `.pkb` file(s).")
            for file_name in pkb_files:
                file_path = os.path.join(PKB_FOLDER, file_name)
                try:
                    with open(file_path, "r") as file:
                        file_content = file.read()

                    # Display file content in an expander
                    with st.expander(f"File: {file_name}"):
                        st.code(file_content, language="sql")
                except Exception as e:
                    st.error(f"Error reading {file_name}: {e}")
        else:
            st.warning("No `.pkb` files found. Please extract files first.")

    # Tab 3: Generate Documentation
    with tab3:
        st.header("Generate Documentation")
        if st.session_state["extracted_files"]:
            st.session_state["enable_doc_explain"] = st.checkbox("Enable Document Explain", value=st.session_state["enable_doc_explain"])

            if st.session_state["enable_doc_explain"]:
                col1, col2 = st.columns([1, 3])
                with col1:
                    st.session_state["selected_option"] = st.radio(
                        "Select the operation:",
                        options=[
                            "Function Document overview",
                            "Functional Document",
                            "Business Document",
                            "Technical Document",
                            "PLSQL Code Explain",
                            "Identify Duplicate and Redundant Code",
                            "Generate Java OO Design​",
                        ],
                        horizontal=False,
                        key="operation_radio",
                    )
                with col2:
                    responses = process_pkb_files(PKB_FOLDER, st.session_state["selected_option"])
                    if responses:
                        for file_name, response in responses.items():
                            st.write(f"**Procedure Name:** `{file_name}`")
                            st.write(response, language="markdown")
        else:
            st.warning("No `.pkb` files found to process. Please extract files first.")

if __name__ == "__main__":
    main()
