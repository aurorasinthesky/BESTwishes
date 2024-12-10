!pip install streamlit
!pip install transformers
!pip install pillow
!pip install fpdf
!pip install requests
import streamlit as st
from transformers import pipeline
from PIL import Image
import io
import requests
import os
from fpdf import FPDF

# Initialize AI Models
@st.cache_resource
def load_text_generator():
    return pipeline("text-generation", model="gpt-3.5-turbo")

def generate_wish_message(prompt, model):
    response = model(prompt, max_length=200, num_return_sequences=1)
    return response[0]['generated_text']

def generate_visual(wish_message, style):
    dalle_prompt = f"{style} style image for the theme: '{wish_message}'"
    response = requests.post(
        "https://api.dalle-mini.ai/v1/generate",
        json={"prompt": dalle_prompt}
    )
    if response.status_code == 200:
        image_data = response.json()["generated_image"]
        return Image.open(io.BytesIO(image_data))
    else:
        return None

def create_pdf(wish_message, image):
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font("Arial", size=12)
    pdf.multi_cell(0, 10, wish_message)
    img_path = "temp_image.png"
    image.save(img_path)
    pdf.image(img_path, x=10, y=50, w=100)
    os.remove(img_path)
    pdf_output = io.BytesIO()
    pdf.output(pdf_output, "F")
    pdf_output.seek(0)
    return pdf_output

# App Interface
st.title("Good Wishes Generator")

# Step 1: Select a Wish Theme
theme = st.selectbox(
    "Choose the occasion:",
    ["New Year", "Holiday Greeting", "Job Congratulations", "Birthday", "Graduation", "Anniversary", "Other"]
)

# Step 2: Provide Recipient Details
recipient = st.text_input("Who is this for? (Name or Relation)")

# Step 3: Custom Prompt for the Wish
user_prompt = st.text_area("Describe how you'd like the message to be (e.g., humorous, heartfelt, formal):")

# Generate the Wish Message
if st.button("Generate Wish Message"):
    if recipient and user_prompt:
        model = load_text_generator()
        prompt = f"Write a {theme} wish for {recipient}. {user_prompt}"
        wish_message = generate_wish_message(prompt, model)
        st.success("Here is your generated message:")
        st.write(wish_message)
        st.session_state["wish_message"] = wish_message
    else:
        st.error("Please provide both recipient and a prompt.")

# Visualize the Wish
if "wish_message" in st.session_state:
    wish_message = st.session_state["wish_message"]
    style = st.selectbox("Choose a visual style:", ["Sketch", "Realistic", "Artistic", "Cartoon", "Abstract", "Pastel", "Fresco"])
    if st.button("Generate Visual"):
        with st.spinner("Generating visual..."):
            visual = generate_visual(wish_message, style)
            if visual:
                st.image(visual, caption="Generated Visual")
                st.session_state["visual"] = visual
            else:
                st.error("Failed to generate visual. Please try again.")

# Combine and Download PDF
if "wish_message" in st.session_state and "visual" in st.session_state:
    if st.button("Download as PDF"):
        pdf_file = create_pdf(st.session_state["wish_message"], st.session_state["visual"])
        st.download_button(
            label="Download Your Wish Card",
            data=pdf_file,
            file_name="wish_card.pdf",
            mime="application/pdf"
        )
