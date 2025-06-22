# SVG to Live2D Asset Extraction Guide - Llama Project

## Overview
Extract individual PNG layers from our Three LLaMa Moon SVG designs for Live2D Cubism workflow. This guide focuses on the single interactive llama character (the Three LLaMa Moon design serves as intro/branding).

**Target**: Production-ready PNG assets optimized for NOM-Network's Melba-Toaster framework.

## Quick Start (Recommended Path)

### 🚀 Method 1: Automated Extraction
**Best for**: Fast iteration, consistent results, batch processing

```bash
# Install Inkscape (if not already installed)
# Windows: Download from inkscape.org
# Mac: brew install inkscape
# Linux: sudo apt install inkscape

# Create extraction script
cat > extract_llama_layers.sh << 'EOF'
#!/bin/bash
echo "Extracting Live2D Llama assets..."

# Core body parts
inkscape --export-id=body --export-id-only --export-png=assets/body.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=neck --export-id-only --export-png=assets/neck.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=head_base --export-id-only --export-png=assets/head_base.png --export-dpi=300 live2d_llama.svg

# Facial features
inkscape --export-id=ear_left --export-id-only --export-png=assets/ear_left.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=ear_right --export-id-only --export-png=assets/ear_right.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=eye_left --export-id-only --export-png=assets/eye_left.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=eye_right --export-id-only --export-png=assets/eye_right.png --export-dpi=300 live2d_llama.svg

# Expression variants
inkscape --export-id=mouth_neutral --export-id-only --export-png=assets/mouth_neutral.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=mouth_open --export-id-only --export-png=assets/mouth_open.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=mouth_smile --export-id-only --export-png=assets/mouth_smile.png --export-dpi=300 live2d_llama.svg
inkscape --export-id=mouth_surprised --export-id-only --export-png=assets/mouth_surprised.png --export-dpi=300 live2d_llama.svg

# Accessories and effects
inkscape --export-id=fluff_overlay --export-id-only --export-png=assets/fluff_overlay.png --export-dpi=300 live2d_llama.svg

echo "Extraction complete! Check assets/ directory"
ls -la assets/
EOF

chmod +x extract_llama_layers.sh
mkdir -p assets
./extract_llama_layers.sh
```

**Success Check**: You should have 11 PNG files in assets/ directory, each 2048x2048px with transparent backgrounds.
```bash
# Extract individual layers by ID
inkscape --export-id=body --export-id-only --export-png=body.png --export-dpi=300 llama.svg
inkscape --export-id=neck --export-id-only --export-png=neck.png --export-dpi=300 llama.svg
inkscape --export-id=head_base --export-id-only --export-png=head_base.png --export-dpi=300 llama.svg

# Batch extraction script (save as extract_layers.sh)
#!/bin/bash
LAYERS=("body" "neck" "head_base" "ear_left" "ear_right" "eye_left" "eye_right" "eyelid_left" "eyelid_right" "snout" "mouth_neutral" "mouth_open" "mouth_smile" "mouth_surprised" "fluff_overlay")

for layer in "${LAYERS[@]}"; do
    inkscape --export-id=$layer --export-id-only --export-png=assets/${layer}.png --export-dpi=300 live2d_llama.svg
    echo "Exported: ${layer}.png"
done
```

### Using Node.js (Advanced)
```javascript
// svg-to-layers.js
const { createSVGWindow } = require('svgdom');
const { SVG, registerWindow } = require('@svgdotjs/svg.js');
const sharp = require('sharp');

// Register SVG.js with node
const window = createSVGWindow();
const document = window.document;
registerWindow(window, document);

async function extractLayers(svgPath) {
    const layerIds = [
        'body', 'neck', 'head_base', 'ear_left', 'ear_right',
        'eye_left', 'eye_right', 'snout', 'mouth_neutral'
    ];
    
    for (const layerId of layerIds) {
        // Extract individual layer
        const svg = SVG(document.documentElement);
        const layer = svg.findOne(`#${layerId}`);
        
        if (layer) {
            const svgString = layer.svg();
            const buffer = Buffer.from(svgString);
            
            await sharp(buffer)
                .resize(2048, 2048)
                .png()
                .toFile(`assets/${layerId}.png`);
                
            console.log(`Extracted: ${layerId}.png`);
        }
    }
}

extractLayers('live2d_llama.svg');
```

## Method 2: Manual Extraction (Photoshop/GIMP)

### Photoshop Workflow
1. **Import SVG**: File → Import → SVG, set to 2048x2048px
2. **Layer Organization**: Each SVG group becomes a layer group
3. **Export Process**:
   ```
   For each layer:
   1. Hide all other layers
   2. File → Export → Export As
   3. Format: PNG
   4. Transparency: Enabled
   5. Resolution: 300 DPI
   6. Name: [layer_id].png
   ```

### GIMP Workflow
1. **Import**: Open SVG, set size to 2048x2048px
2. **Layer Management**: Use Layers panel to toggle visibility
3. **Batch Export**:
   - Use File → Export Layers plugin
   - Configure PNG settings with transparency
   - Set naming pattern to layer names

## ✅ Asset Validation Checklist

### Technical Requirements Verification
```bash
# Run this validation script after extraction
cat > validate_assets.sh << 'EOF'
#!/bin/bash
echo "🔍 Validating Live2D assets..."

required_files=("body.png" "neck.png" "head_base.png" "ear_left.png" "ear_right.png" "eye_left.png" "eye_right.png" "mouth_neutral.png")
missing_count=0

for file in "${required_files[@]}"; do
    if [[ -f "assets/$file" ]]; then
        size=$(identify -format "%wx%h" "assets/$file" 2>/dev/null || echo "unknown")
        filesize=$(ls -lh "assets/$file" | awk '{print $5}')
        echo "✅ $file ($size, $filesize)"
    else
        echo "❌ Missing: $file"
        ((missing_count++))
    fi
done

if [[ $missing_count -eq 0 ]]; then
    echo "🎉 All assets validated! Ready for Live2D Cubism import."
else
    echo "⚠️  $missing_count files missing. Check extraction process."
fi
EOF

chmod +x validate_assets.sh
./validate_assets.sh
```

### Quality Checklist
- [ ] **Resolution**: All files 2048x2048px (check with `identify assets/*.png`)
- [ ] **Transparency**: Backgrounds fully transparent (no white/colored backgrounds)
- [ ] **Alignment**: Parts stack correctly when overlaid (test in image editor)
- [ ] **Edge Quality**: Clean anti-aliasing, no pixelation at 100% zoom
- [ ] **File Size**: Each file 200KB-2MB (efficient but not over-compressed)

### Educational-Specific Validation
- [ ] **Expression Range**: Mouth variants show clear differences (neutral → open → smile → surprised)
- [ ] **Eye Contact**: Eyes positioned for natural student engagement
- [ ] **Ear Expressiveness**: Clear difference between alert/relaxed ear positions
- [ ] **Teaching Pose**: Head tilt and body language appropriate for instruction

**Ready for Live2D**: When all checkboxes are complete, import assets into Cubism Editor following the Live2D Pipeline Guide.

## File Naming Convention
```
body.png              # Main torso
neck.png              # Neck section
head_base.png         # Head without features
ear_left.png          # Left ear
ear_right.png         # Right ear
eye_left.png          # Left eye complete
eye_right.png         # Right eye complete
eyelid_left.png       # Left eyelid
eyelid_right.png      # Right eyelid
snout.png             # Nose/snout area
mouth_neutral.png     # Default mouth
mouth_open.png        # Open mouth
mouth_smile.png       # Smiling mouth
mouth_surprised.png   # Surprised expression
fluff_overlay.png     # Additional fluff details
```

## Asset Preparation for Live2D

### Canvas Padding
Each exported PNG needs consistent canvas size:
- **Working area**: Center 1500x1500px
- **Padding**: 274px on all sides
- **Total canvas**: 2048x2048px

### Alpha Channel Optimization
```python
# Python script to optimize PNG alpha channels
from PIL import Image
import os

def optimize_alpha(input_path, output_path):
    img = Image.open(input_path).convert("RGBA")
    
    # Clean up alpha edges
    data = img.getdata()
    cleaned_data = []
    
    for item in data:
        # If pixel is very transparent, make it fully transparent
        if item[3] < 25:  # Alpha threshold
            cleaned_data.append((0, 0, 0, 0))
        else:
            cleaned_data.append(item)
    
    img.putdata(cleaned_data)
    img.save(output_path, "PNG", optimize=True)

# Process all assets
for filename in os.listdir("assets/"):
    if filename.endswith(".png"):
        optimize_alpha(f"assets/{filename}", f"assets/optimized_{filename}")
```

## Layer-Specific Guidelines

### Eyes (`eye_left.png`, `eye_right.png`)
- Include full eye socket and iris
- Pupil should be centered for neutral gaze
- Add subtle highlight for liveliness
- Maintain symmetry between left and right

### Ears (`ear_left.png`, `ear_right.png`)
- Export with rotation pivot at base
- Include inner ear detail
- Ensure tips have clean edges for animation
- Consider slight asymmetry for character

### Mouth Variants
- **Neutral**: Slight smile, relaxed
- **Open**: Oval shape for speech
- **Smile**: Pronounced upward curve
- **Surprised**: Small circular opening

### Body Parts
- **Body**: Main torso with breathing deformation zones
- **Neck**: Generous overlap with head and body
- **Fluff**: Secondary animation elements

## Troubleshooting

### Common Issues
1. **Misaligned layers**: Check SVG transform origins
2. **Pixelated exports**: Verify DPI settings (300+ recommended)
3. **Missing transparency**: Ensure alpha channel preservation
4. **Inconsistent sizing**: Use consistent export dimensions

### Validation Script
```bash
#!/bin/bash
# validate_assets.sh

echo "Validating Live2D assets..."

required_files=("body.png" "neck.png" "head_base.png" "ear_left.png" "ear_right.png" "eye_left.png" "eye_right.png")

for file in "${required_files[@]}"; do
    if [[ -f "assets/$file" ]]; then
        size=$(identify -format "%wx%h" "assets/$file")
        echo "✓ $file ($size)"
    else
        echo "✗ Missing: $file"
    fi
done

echo "Asset validation complete."
```

## Next Steps
1. Run extraction process on current SVG llama
2. Validate all assets using checklist
3. Import into Live2D Cubism Editor
4. Begin mesh generation phase

This systematic approach ensures high-quality assets ready for Live2D rigging!