import streamlit as st
from rembg import remove
from PIL import Image, ImageEnhance
import cv2
import numpy as np

st.set_page_config(page_title="AI Photo Tool", layout="centered")

st.title("📸 AI Photo Editor (A4 + Passport Size)")

uploaded_file = st.file_uploader("Upload Photo", type=["jpg", "png", "jpeg"])

def make_a4(image):
    a4_width, a4_height = 2480, 3508
    bg = Image.new("RGB", (a4_width, a4_height), (255, 255, 255))
    image.thumbnail((2000, 3000))
    x = (a4_width - image.width) // 2
    y = (a4_height - image.height) // 2
    bg.paste(image, (x, y))
    return bg

def make_passport(image):
    return image.resize((600, 600))

def passport_on_a4(passport):
    a4 = Image.new("RGB", (2480, 3508), (255, 255, 255))
    x_offset, y_offset = 100, 100
    gap = 50

    for row in range(4):
        for col in range(3):
            x = x_offset + col * (passport.width + gap)
            y = y_offset + row * (passport.height + gap)
            a4.paste(passport, (x, y))

    return a4

if uploaded_file:
    image = Image.open(uploaded_file)
    st.image(image, caption="Original Image")

    if st.button("🚀 One Click Process"):

        output = remove(image)

        img_np = np.array(output)
        img_np = cv2.fastNlMeansDenoisingColored(img_np, None, 10, 10, 7, 21)
        clean_img = Image.fromarray(img_np)

        sharp = ImageEnhance.Sharpness(clean_img).enhance(2.5)
        final = ImageEnhance.Contrast(sharp).enhance(1.8)

        bg = Image.new("RGB", final.size, (255, 255, 255))
        bg.paste(final, mask=final.split()[3] if final.mode == 'RGBA' else None)

        a4_image = make_a4(bg)
        passport = make_passport(bg)
        passport_sheet = passport_on_a4(passport)

        st.image(a4_image, caption="🖨️ A4 Print Ready")
        st.image(passport, caption="🪪 Passport Size")
        st.image(passport_sheet, caption="📄 Passport Sheet")

        a4_image.save("A4.png")
        passport.save("passport.png")
        passport_sheet.save("passport_sheet.png")

        with open("A4.png", "rb") as f:
            st.download_button("📥 Download A4", f, "A4.png")

        with open("passport.png", "rb") as f:
            st.download_button("📥 Download Passport", f, "passport.png")

        with open("passport_sheet.png", "rb") as f:
            st.download_button("📥 Download Sheet", f, "passport_sheet.png")
