# color-identifierimport streamlit as st
import colorsys
import re

st.set_page_config(
    page_title="Color Match & Identifier",
    page_icon="🎨",
    layout="centered"
)

st.title("🎨 Color Match & Identifier")
st.write("Enter a color code to identify the color and generate useful matching colors.")

def hex_to_rgb(hex_color):
    hex_color = hex_color.strip().lstrip("#")
    if len(hex_color) == 3:
        hex_color = "".join(c * 2 for c in hex_color)
    if not re.fullmatch(r"[0-9a-fA-F]{6}", hex_color):
        raise ValueError("Invalid HEX color")
    return tuple(int(hex_color[i:i+2], 16) for i in (0, 2, 4))

def rgb_to_hex(rgb):
    return "#{:02X}{:02X}{:02X}".format(*rgb)

def rgb_to_hsl(rgb):
    r, g, b = [x / 255 for x in rgb]
    h, l, s = colorsys.rgb_to_hls(r, g, b)
    return round(h * 360), round(s * 100), round(l * 100)

def hsl_to_rgb(h, s, l):
    r, g, b = colorsys.hls_to_rgb(h / 360, l / 100, s / 100)
    return tuple(round(x * 255) for x in (r, g, b))

def color_name_simple(rgb):
    r, g, b = rgb
    if max(rgb) - min(rgb) < 18:
        if max(rgb) < 45:
            return "Black / Near Black"
        if min(rgb) > 210:
            return "White / Near White"
        return "Gray"
    if r > 180 and g < 100 and b < 100:
        return "Red"
    if r > 180 and g > 120 and b < 100:
        return "Orange / Warm"
    if r > 180 and g > 160 and b < 100:
        return "Yellow"
    if g > r * 1.25 and g > b * 1.15:
        return "Green"
    if b > r * 1.25 and b > g * 1.05:
        return "Blue"
    if r > 120 and b > 120 and g < 120:
        return "Purple / Magenta"
    if g > 100 and b > 100 and r < 120:
        return "Cyan / Teal"
    return "Mixed / Neutral"

def card(hex_color, label):
    st.markdown(
        f"""
        <div style="
            border:1px solid #ddd;
            border-radius:12px;
            padding:14px;
            margin:6px 0;
            background:{hex_color};
            min-height:70px;">
            <b style="background:white;padding:4px 8px;border-radius:6px;">
                {label}: {hex_color}
            </b>
        </div>
        """,
        unsafe_allow_html=True
    )

mode = st.radio("Input type", ["HEX", "RGB"], horizontal=True)

if mode == "HEX":
    value = st.text_input("Enter HEX code", "#3498DB")
    try:
        rgb = hex_to_rgb(value)
    except ValueError:
        st.error("Please enter a valid HEX code, for example #3498DB.")
        st.stop()
else:
    c1, c2, c3 = st.columns(3)
    r = c1.number_input("R", 0, 255, 52)
    g = c2.number_input("G", 0, 255, 152)
    b = c3.number_input("B", 0, 255, 219)
    rgb = (r, g, b)

hex_code = rgb_to_hex(rgb)
h, s, l = rgb_to_hsl(rgb)

# Matching colors
complementary = hsl_to_rgb((h + 180) % 360, s, l)
analogous_1 = hsl_to_rgb((h + 30) % 360, s, l)
analogous_2 = hsl_to_rgb((h - 30) % 360, s, l)
triadic_1 = hsl_to_rgb((h + 120) % 360, s, l)
triadic_2 = hsl_to_rgb((h + 240) % 360, s, l)

st.subheader("Selected Color")
card(hex_code, "Color")

c1, c2, c3, c4 = st.columns(4)
c1.metric("HEX", hex_code)
c2.metric("RGB", str(rgb))
c3.metric("HSL", f"{h}°, {s}%, {l}%")
c4.metric("Type", color_name_simple(rgb))

st.subheader("Color Matching Palette")
card(rgb_to_hex(complementary), "Complementary")
card(rgb_to_hex(analogous_1), "Analogous 1")
card(rgb_to_hex(analogous_2), "Analogous 2")
card(rgb_to_hex(triadic_1), "Triadic 1")
card(rgb_to_hex(triadic_2), "Triadic 2")

st.info("Tip: HEX and RGB values can be copied and used in websites, graphics, textile design, and other applications.")

st.caption("Academic prototype — Color Match & Identifier")
