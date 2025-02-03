import tkinter as tk
from tkinter import filedialog, messagebox
from PIL import Image, ImageTk
import os
import json
from pdf2image import convert_from_path

PRESETS_FILE = "image_processor_presets.json"

def load_presets():
    if os.path.exists(PRESETS_FILE):
        with open(PRESETS_FILE, "r") as f:
            return json.load(f)
    return {}

def save_presets(presets):
    with open(PRESETS_FILE, "w") as f:
        json.dump(presets, f)

def apply_preset(preset_name):
    presets = load_presets()
    preset = presets.get(preset_name)
    if not preset:
        messagebox.showerror("Error", f"Preset '{preset_name}' not found.")
        return

    resize_option.set(preset['resize_option'])
    entry_resize_percentage.delete(0, tk.END)
    entry_resize_percentage.insert(0, preset['resize_percentage'])
    entry_width.delete(0, tk.END)
    entry_width.insert(0, preset['width'])
    entry_height.delete(0, tk.END)
    entry_height.insert(0, preset['height'])

    slider_quality.set(preset['quality'])
    var_webp.set(preset['convert_to_webp'])
    entry_new_name.delete(0, tk.END)
    entry_new_name.insert(0, preset['new_name'])

def save_current_preset():
    preset_name = entry_preset_name.get().strip()
    if not preset_name:
        messagebox.showerror("Error", "Please enter a preset name.")
        return

    preset = {
        "resize_option": resize_option.get(),
        "resize_percentage": entry_resize_percentage.get(),
        "width": entry_width.get(),
        "height": entry_height.get(),
        "quality": slider_quality.get(),
        "convert_to_webp": var_webp.get(),
        "new_name": entry_new_name.get()
    }

    presets = load_presets()
    presets[preset_name] = preset
    save_presets(presets)

    messagebox.showinfo("Success", f"Preset '{preset_name}' saved.")

def open_crop_window(img):
    crop_window = tk.Toplevel(root)
    crop_window.title("Crop Image")
    img_copy = img.copy()
    tk_image = ImageTk.PhotoImage(img_copy)
    canvas = tk.Canvas(crop_window, width=img_copy.width, height=img_copy.height)
    canvas.create_image(0, 0, anchor=tk.NW, image=tk_image)
    canvas.image = tk_image
    canvas.pack()

    crop_rect = [0, 0, 0, 0]
    rect_id = None

    def start_crop(event):
        crop_rect[0] = event.x
        crop_rect[1] = event.y

    def update_crop_rectangle(event):
        nonlocal rect_id
        crop_rect[2] = event.x
        crop_rect[3] = event.y
        if rect_id:
            canvas.delete(rect_id)
        rect_id = canvas.create_rectangle(crop_rect[0], crop_rect[1], crop_rect[2], crop_rect[3], outline="red")

    def finalize_crop():
        left, top, right, bottom = map(int, (min(crop_rect[0], crop_rect[2]), min(crop_rect[1], crop_rect[3]),
                                             max(crop_rect[0], crop_rect[2]), max(crop_rect[1], crop_rect[3])))
        # Ensure the coordinates are within image boundaries
        left = max(0, left)
        top = max(0, top)
        right = min(img_copy.width, right)
        bottom = min(img_copy.height, bottom)
        if left < right and top < bottom:
            cropped_img = img.crop((left, top, right, bottom))
            process_images.cropped_image = cropped_img
        else:
            messagebox.showerror("Error", "Invalid crop area selected.")
        crop_window.destroy()

    canvas.bind("<Button-1>", start_crop)
    canvas.bind("<B1-Motion>", update_crop_rectangle)
    crop_button = tk.Button(crop_window, text="Crop", command=finalize_crop)
    crop_button.pack()
    crop_window.mainloop()

    return getattr(process_images, 'cropped_image', img)

def process_images(input_paths, output_dir, new_name="", resize_option=1,
                   resize_percentage=100, width=None, height=None, quality=85, convert_to_webp=True, crop=False):
    try:
        for input_path in input_paths:
            original_name = os.path.splitext(os.path.basename(input_path))[0]
            file_ext = os.path.splitext(input_path)[1].lower()
            if file_ext == '.pdf':
                # Convert PDF pages to images
                pages = convert_from_path(input_path, dpi=300)
                for page_number, img in enumerate(pages, start=1):
                    if crop:
                        img = open_crop_window(img)

                    img = resize_image(img, resize_option, resize_percentage, width, height)

                    if new_name:
                        page_name = f"{new_name}_{page_number}"
                    else:
                        page_name = f"{original_name}_{page_number}"

                    save_image(img, output_dir, page_name, quality, convert_to_webp)
                    print(f"Processed page {page_number} of PDF saved to: {output_dir}")
            else:
                with Image.open(input_path) as img:
                    if crop:
                        img = open_crop_window(img)

                    img = resize_image(img, resize_option, resize_percentage, width, height)

                    save_image(img, output_dir, new_name, quality, convert_to_webp, original_name)
                    print(f"Processed image saved to: {output_dir}")

        messagebox.showinfo("Success", "All images have been processed and saved.")
    except Exception as e:
        messagebox.showerror("Error", f"Something went wrong: {e}")


def resize_image(img, resize_option, resize_percentage, width, height):
    if resize_option == 1:
        # Resize by percentage
        resize_percentage = float(resize_percentage)
        width_orig, height_orig = img.size
        new_size = (int(width_orig * resize_percentage / 100), int(height_orig * resize_percentage / 100))
    else:
        # Resize to specific dimensions
        new_size = (int(width), int(height))

    return img.resize(new_size, Image.LANCZOS)

def save_image(img, output_dir, new_name, quality, convert_to_webp, original_name=""):
    if convert_to_webp:
        output_format = 'WEBP'
        extension = '.webp'
    else:
        output_format = img.format if img.format else 'PNG'
        extension = f".{output_format.lower()}"

    if new_name:
        output_filename = f"{new_name}{extension}"
    elif original_name:
        output_filename = f"{original_name}{extension}"
    else:
        output_filename = f"image{extension}"

    output_path = os.path.join(output_dir, output_filename)

    # If the file already exists, append a counter to the filename
    counter = 1
    base_name, ext = os.path.splitext(output_filename)
    while os.path.exists(output_path):
        output_filename = f"{base_name}_{counter}{extension}"
        output_path = os.path.join(output_dir, output_filename)
        counter += 1

    if os.path.exists(output_path):
        print(f"Warning: {output_path} already exists and will be overwritten.")

    img.save(output_path, format=output_format, quality=quality)


def browse_files():
    file_types = [
        ("Image and PDF files", "*.png *.jpg *.jpeg *.bmp *.gif *.tiff *.pdf"),
        ("All files", "*.*")
    ]
    file_paths = filedialog.askopenfilenames(filetypes=file_types)
    if file_paths:
        entry_input_path.delete(0, tk.END)
        entry_input_path.insert(0, ','.join(file_paths))


def browse_directory():
    directory = filedialog.askdirectory()
    if directory:
        entry_output_dir.delete(0, tk.END)
        entry_output_dir.insert(0, directory)

def start_processing(crop=False):
    input_paths = entry_input_path.get().split(',')
    output_dir = entry_output_dir.get()
    new_name = entry_new_name.get()
    quality = slider_quality.get()
    convert_to_webp = var_webp.get()
    selected_resize_option = resize_option.get()

    if not input_paths or not output_dir:
        messagebox.showerror("Error", "Input files and output directory are required.")
        return

    resize_percentage = None
    width = None
    height = None

    if selected_resize_option == 1:
        # Resize by percentage
        resize_percentage = entry_resize_percentage.get()
    else:
        # Resize to specific dimensions
        width = entry_width.get()
        height = entry_height.get()

        if not width or not height:
            messagebox.showerror("Error", "Width and height are required for resizing to specific dimensions.")
            return

    process_images(input_paths, output_dir, new_name, selected_resize_option,
                   resize_percentage, width, height, quality, convert_to_webp, crop)

def update_resize_inputs():
    if resize_option.get() == 1:
        # Enable percentage entry, disable width and height entries
        entry_resize_percentage.config(state='normal')
        entry_width.config(state='disabled')
        entry_height.config(state='disabled')
    else:
        # Disable percentage entry, enable width and height entries
        entry_resize_percentage.config(state='disabled')
        entry_width.config(state='normal')
        entry_height.config(state='normal')

def update_preset_dropdown():
    preset_names = list(load_presets().keys())
    preset_var.set(preset_names[0] if preset_names else "")
    menu = preset_dropdown["menu"]
    menu.delete(0, "end")
    for name in preset_names:
        menu.add_command(label=name, command=lambda value=name: preset_var.set(value))

# Setting up the GUI
root = tk.Tk()
root.title("Image Processor with PDF Support")

# Input files
tk.Label(root, text="Input images:").grid(row=0, column=0, padx=10, pady=10, sticky="e")
entry_input_path = tk.Entry(root, width=50)
entry_input_path.grid(row=0, column=1, padx=10, pady=10)
tk.Button(root, text="Browse", command=browse_files).grid(row=0, column=2, padx=10, pady=10)

# Output directory
tk.Label(root, text="Output directory:").grid(row=1, column=0, padx=10, pady=10, sticky="e")
entry_output_dir = tk.Entry(root, width=50)
entry_output_dir.grid(row=1, column=1, padx=10, pady=10)
tk.Button(root, text="Browse", command=browse_directory).grid(row=1, column=2, padx=10, pady=10)

# New name
tk.Label(root, text="New file name:").grid(row=2, column=0, padx=10, pady=10, sticky="e")
entry_new_name = tk.Entry(root, width=50)
entry_new_name.grid(row=2, column=1, padx=10, pady=10)

# Resize options
resize_option = tk.IntVar(value=1)

tk.Label(root, text="Resize options:").grid(row=3, column=0, padx=10, pady=10, sticky="ne")
frame_resize_options = tk.Frame(root)
frame_resize_options.grid(row=3, column=1, padx=10, pady=10, sticky="w")

radio_percentage = tk.Radiobutton(frame_resize_options, text="By Percentage", variable=resize_option, value=1, command=update_resize_inputs)
radio_percentage.grid(row=0, column=0, sticky="w")

entry_resize_percentage = tk.Entry(frame_resize_options, width=10)
entry_resize_percentage.insert(0, "100")
entry_resize_percentage.grid(row=0, column=1, padx=5)

tk.Label(frame_resize_options, text="%").grid(row=0, column=2)

radio_dimensions = tk.Radiobutton(frame_resize_options, text="To Dimensions (px)", variable=resize_option, value=2, command=update_resize_inputs)
radio_dimensions.grid(row=1, column=0, sticky="w")

tk.Label(frame_resize_options, text="Width:").grid(row=1, column=1, sticky="e")
entry_width = tk.Entry(frame_resize_options, width=5)
entry_width.grid(row=1, column=2, padx=5)

tk.Label(frame_resize_options, text="Height:").grid(row=1, column=3, sticky="e")
entry_height = tk.Entry(frame_resize_options, width=5)
entry_height.grid(row=1, column=4, padx=5)

# Initialize the state of resize inputs
update_resize_inputs()

# Quality (compression)
tk.Label(root, text="Quality:").grid(row=4, column=0, padx=10, pady=10, sticky="e")
slider_quality = tk.Scale(root, from_=1, to=100, orient=tk.HORIZONTAL)
slider_quality.set(85)
slider_quality.grid(row=4, column=1, padx=10, pady=10, sticky="w")

# WebP conversion
var_webp = tk.BooleanVar()
var_webp.set(True)
tk.Checkbutton(root, text="Convert to WebP", variable=var_webp).grid(row=5, column=1, padx=10, pady=10, sticky="w")

# Crop button
tk.Button(root, text="Crop Image", command=lambda: start_processing(crop=True)).grid(row=6, column=1, padx=10, pady=10, sticky="w")

# Preset name
tk.Label(root, text="Preset name:").grid(row=7, column=0, padx=10, pady=10, sticky="e")
entry_preset_name = tk.Entry(root, width=50)
entry_preset_name.grid(row=7, column=1, padx=10, pady=10)

# Save preset button
tk.Button(root, text="Save Preset", command=save_current_preset).grid(row=8, column=1, padx=10, pady=10, sticky="w")

# Load preset dropdown
tk.Label(root, text="Load Preset:").grid(row=9, column=0, padx=10, pady=10, sticky="e")
preset_var = tk.StringVar(root)
preset_names = list(load_presets().keys())
preset_var.set(preset_names[0] if preset_names else "")
preset_dropdown = tk.OptionMenu(root, preset_var, *preset_names)
preset_dropdown.grid(row=9, column=1, padx=10, pady=10, sticky="w")
tk.Button(root, text="Apply Preset", command=lambda: apply_preset(preset_var.get())).grid(row=9, column=2, padx=10, pady=10)

# Update preset dropdown menu
def on_preset_menu_click(event):
    update_preset_dropdown()

preset_dropdown.bind("<Button-1>", on_preset_menu_click)

# Start button
tk.Button(root, text="Start", command=lambda: start_processing()).grid(row=10, column=1, padx=10, pady=10)

root.mainloop()
