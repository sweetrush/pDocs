# Using Streamlit 
## About Streamlit 
- This page provides some information about using streamlit the python libary for data support and manipulation and virtualization. 

- To Use the streamlit you need to install it using **pip** the python install package manager
```bash
pip install streamlit 
```
- Streamlit code start with the following at the top of the Python Script 
```python
from openai import OpenAI
# Add your other additional code to

# End of Code 
```

## Using Colors for Text in Streamlit 
```python 

st.text_input(":green[Name]", value="", max_chars=None) # This will make the Lable in Green Color 
st.text_input(":blue[Name]", value="", max_chars=None)  # This will make the label in Blue Color 
st.text_input(":red[Name]", value="", max_chars=None)   # This will make the label in Red Color 

```

## Creating Pages 
```python 
st.page_link()  # this is the new Page upade for Streamlit.

```