# Kimberlite_Finder_Dr.Mutlu-Zeybek

"""
ZEYBEK-3 Model: Geological Map with Distinct Lithology Colors
COMPLETE LEGEND with All Units, Faults, and Structural Elements
FIXED: Regular fault sequence F1-F24 properly displayed
"""

import matplotlib.pyplot as plt
import matplotlib.patches as patches
from matplotlib.patches import Rectangle, Circle
import numpy as np

# Set up the figure
fig, ax = plt.subplots(1, 1, figsize=(16, 13))
ax.set_xlim(-1.5, 8.5)
ax.set_ylim(-1.5, 29)
ax.set_aspect("equal")

# ============================================================
# DISTINCT COLOR SCHEME FOR EACH LITHOLOGICAL UNIT (L1-L13)
# ============================================================
color_scheme = {
    "L1": "#FF0000",  # Epiclastics - Pure RED
    "L2": "#FF8C00",  # Tuff Ring - Dark ORANGE
    "L3": "#FFD700",  # Pyroclastics - GOLD
    "L4": "#00CED1",  # Sandstone - Dark TURQUOISE
    "L5": "#4169E1",  # Shale 1 - Royal BLUE
    "L6": "#32CD32",  # Sill - Lime GREEN
    "L7": "#8A2BE2",  # Shale 2 - Blue VIOLET
    "L8": "#FF69B4",  # Sandstone - Hot PINK
    "L9": "#CD853F",  # Tillite - Peru BROWN
    "L10": "#2F4F4F",  # Quartzite - Dark SLATE GRAY
    "L11": "#DAA520",  # Schist - Goldenrod
    "L12": "#C0C0C0",  # Gneiss - SILVER
    "L13": "#A0522D",  # Granite - Sienna BROWN
}

# Define the coordinate system
x0, x1, x2 = 0, 3, 6
y = list(range(28))  # y0 to y27

# Define lithology sequence: (y_bottom, y_top, label)
lithology_sequence = [
    (y[1], y[2], "L13"),  # Granite
    (y[2], y[3], "L12"),  # Gneiss
    (y[3], y[4], "L11"),  # Schist
    (y[4], y[5], "L10"),  # Quartzite
    (y[5], y[6], "L9"),  # Tillite
    (y[6], y[7], "L8"),  # Sandstone
    (y[7], y[8], "L7"),  # Shale 2
    (y[8], y[9], "L6"),  # Sill
    (y[9], y[10], "L5"),  # Shale 1
    (y[10], y[11], "L4"),  # Sandstone
    (y[11], y[12], "L3"),  # Pyroclastics
    (y[12], y[13], "L2"),  # Tuff Ring (below L1)
    (y[13], y[15], "L1"),  # Epiclastics (KIMBERLITE CRATER FACIES)
    (y[15], y[16], "L2"),  # Tuff Ring (above L1)
    (y[16], y[17], "L3"),  # Pyroclastics
    (y[17], y[18], "L4"),  # Sandstone
    (y[18], y[19], "L5"),  # Shale 1
    (y[19], y[20], "L6"),  # Sill
    (y[20], y[21], "L7"),  # Shale 2
    (y[21], y[22], "L8"),  # Sandstone
    (y[22], y[23], "L9"),  # Tillite
    (y[23], y[24], "L10"),  # Quartzite
    (y[24], y[25], "L11"),  # Schist
    (y[25], y[26], "L12"),  # Gneiss
    (y[26], y[27], "L13"),  # Granite
]


# Function to draw a lithological unit
def draw_lithology(y_bottom, y_top, label, alpha=0.85):
    height = y_top - y_bottom
    color = color_scheme.get(label, "#CCCCCC")
    rect = Rectangle(
        (x0, y_bottom),
        x2 - x0,
        height,
        facecolor=color,
        edgecolor="black",
        linewidth=1.5,
        alpha=alpha,
    )
    ax.add_patch(rect)

    center_x = x0 + (x2 - x0) / 2
    center_y = y_bottom + height / 2

    if label == "L1":
        ax.text(
            center_x,
            center_y,
            f"{label}\n(Epiclastics)",
            ha="center",
            va="center",
            fontsize=11,
            fontweight="bold",
            color="white",
            bbox=dict(boxstyle="round,pad=0.3", facecolor="black", alpha=0.7),
        )
    else:
        if label in ["L4", "L5", "L10", "L1"]:
            text_color = "white"
        else:
            text_color = "black"
        ax.text(
            center_x,
            center_y,
            label,
            ha="center",
            va="center",
            fontsize=10,
            fontweight="bold",
            color=text_color,
        )


# Draw all lithological units
for y_bottom, y_top, label in lithology_sequence:
    draw_lithology(y_bottom, y_top, label)

# ============================================================
# HIGHLIGHT L1 (Epiclastics) with Special Border
# ============================================================
l1_y_bottom = y[13]
l1_y_top = y[15]
l1_rect = Rectangle(
    (x0, l1_y_bottom),
    x2 - x0,
    l1_y_top - l1_y_bottom,
    facecolor="none",
    edgecolor="#FF0000",
    linewidth=4,
    linestyle="--",
)
ax.add_patch(l1_rect)

# ============================================================
# DRAW KIMBERLITE PIPE (K) at Centroid (x1, y14)
# ============================================================
k_x = x1
k_y = y[14]
k_radius = 0.5

k_circle = Circle(
    (k_x, k_y),
    k_radius,
    facecolor="#8B0000",
    edgecolor="#FF0000",
    linewidth=3,
    zorder=10,
)
ax.add_patch(k_circle)
ax.text(
    k_x,
    k_y,
    "K\n💎",
    ha="center",
    va="center",
    fontsize=14,
    fontweight="bold",
    color="white",
)

# ============================================================
# DRAW FAULT NETWORK (F1-F24) - REGULAR SEQUENCE
# ============================================================
fault_color = "#1E3A8A"

# ============================================================
# FIX: Draw ALL faults F1-F24 in regular sequence
# ============================================================
for i in range(1, 25):  # F1 through F24
    y_pos = y[27 - i]  # F1 at top (y=26), F24 at bottom (y=1)

    # Determine fault style based on position
    if i in [1, 12, 13, 24]:  # Major faults
        linewidth = 3.5
        linestyle = "-"
        fontsize = 11
        alpha = 1.0
        label_offset = 0.3
    else:  # Intermediate faults
        linewidth = 1.5
        linestyle = "--"
        fontsize = 8
        alpha = 0.6
        label_offset = 0.3

    # Draw the fault line
    ax.plot(
        [x0, x2],
        [y_pos, y_pos],
        color=fault_color,
        linewidth=linewidth,
        linestyle=linestyle,
    )

    # Add label on the right side
    ax.text(
        x2 + label_offset,
        y_pos,
        f"F{i}",
        fontsize=fontsize,
        fontweight="bold" if i in [1, 12, 13, 24] else "normal",
        color=fault_color,
        alpha=alpha,
        va="center",
    )

# ============================================================
# DRAW THE FAULT-BOUNDED COMPARTMENT
# ============================================================
compartment = Rectangle(
    (x0, l1_y_bottom),
    x2 - x0,
    l1_y_top - l1_y_bottom,
    facecolor="green",
    edgecolor="#00AA00",
    linewidth=3,
    linestyle="-",
    alpha=0.08,
)
ax.add_patch(compartment)

# ============================================================
# ADD X AND Y COORDINATE AXES
# ============================================================

# X-Axis
ax.axhline(y=-0.5, color="black", linewidth=2, zorder=5)
ax.annotate(
    "",
    xy=(7.5, -0.5),
    xytext=(-0.5, -0.5),
    arrowprops=dict(arrowstyle="->", color="black", linewidth=2),
)
ax.text(7.8, -0.5, "X", fontsize=14, fontweight="bold", ha="center", va="center")

# X-axis ticks
x_ticks = [x0, x1, x2]
x_labels = ["x₀ = 0", "x₁ = 3", "x₂ = 6"]
for xtick, xlabel in zip(x_ticks, x_labels):
    ax.plot([xtick, xtick], [-0.7, -0.3], color="black", linewidth=1.5)
    ax.text(xtick, -1.2, xlabel, ha="center", va="top", fontsize=10, fontweight="bold")

# Y-Axis
ax.axvline(x=-0.5, color="black", linewidth=2, zorder=5)
ax.annotate(
    "",
    xy=(-0.5, 28),
    xytext=(-0.5, -0.5),
    arrowprops=dict(arrowstyle="->", color="black", linewidth=2),
)
ax.text(-1.0, 28.5, "Y", fontsize=14, fontweight="bold", ha="center", va="center")

# Y-axis ticks
y_ticks = [y[0], y[5], y[10], y[13], y[14], y[15], y[20], y[25], y[27]]
y_labels = [
    "y₀=0",
    "y₅=5",
    "y₁₀=10",
    "y₁₃=13",
    "y₁₄=14",
    "y₁₅=15",
    "y₂₀=20",
    "y₂₅=25",
    "y₂₇=27",
]
for ytick, ylabel in zip(y_ticks, y_labels):
    ax.plot([-0.7, -0.3], [ytick, ytick], color="black", linewidth=1.5)
    ax.text(-1.2, ytick, ylabel, ha="right", va="center", fontsize=9, fontweight="bold")

# Grid
for xtick in np.linspace(0, 6, 13):
    ax.axvline(
        x=xtick, ymin=0, ymax=1, color="gray", linewidth=0.3, alpha=0.15, zorder=1
    )
for ytick in range(0, 28):
    ax.axhline(
        y=ytick, xmin=0, xmax=1, color="gray", linewidth=0.3, alpha=0.15, zorder=1
    )

# ============================================================
# COMPREHENSIVE LEGEND
# ============================================================

# Create legend elements - LITHOLOGICAL UNITS
legend_elements = []

# Add L1-L13 with their colors
for label in [
    "L1",
    "L2",
    "L3",
    "L4",
    "L5",
    "L6",
    "L7",
    "L8",
    "L9",
    "L10",
    "L11",
    "L12",
    "L13",
]:
    color = color_scheme.get(label, "#CCCCCC")
    name_map = {
        "L1": "Epiclastics (Kimberlite Crater) ★",
        "L2": "Tuff Ring",
        "L3": "Pyroclastics",
        "L4": "Sandstone",
        "L5": "Shale 1",
        "L6": "Sill",
        "L7": "Shale 2",
        "L8": "Sandstone",
        "L9": "Tillite",
        "L10": "Quartzite",
        "L11": "Schist",
        "L12": "Gneiss",
        "L13": "Granite",
    }
    patch = patches.Patch(
        facecolor=color,
        edgecolor="black",
        label=f"{label}: {name_map.get(label, label)}",
    )
    legend_elements.append(patch)

# Add STRUCTURAL ELEMENTS
legend_elements.append(
    patches.Patch(
        facecolor="none",
        edgecolor=fault_color,
        linewidth=3.5,
        label="Major Faults (F1, F12, F13, F24)",
    )
)
legend_elements.append(
    patches.Patch(
        facecolor="none",
        edgecolor=fault_color,
        linewidth=1.5,
        linestyle="--",
        label="Intermediate Faults (F2-F11, F14-F23)",
    )
)
legend_elements.append(
    patches.Patch(
        facecolor="green",
        alpha=0.15,
        edgecolor="#00AA00",
        linewidth=3,
        label="Fault-Bounded Compartment",
    )
)
legend_elements.append(
    patches.Patch(
        facecolor="none",
        edgecolor="#FF0000",
        linewidth=4,
        linestyle="--",
        label="L1 Boundary (Epiclastics)",
    )
)

# Add TARGET ELEMENT
legend_elements.append(
    patches.Patch(
        facecolor="#8B0000",
        edgecolor="#FF0000",
        label="K: Kimberlite Pipe (Diamond) 💎",
    )
)

# Add COORDINATE SYSTEM
legend_elements.append(
    patches.Patch(
        facecolor="yellow",
        alpha=0.3,
        edgecolor="orange",
        linewidth=2,
        label="Centroid Point (x₁, y₁₄)",
    )
)

# ============================================================
# CREATE THE LEGEND
# ============================================================

legend = ax.legend(
    handles=legend_elements,
    loc="center left",
    bbox_to_anchor=(1.02, 0.4),
    fontsize=8,
    title="LEGEND",
    title_fontsize=12,
    framealpha=0.95,
    edgecolor="black",
    facecolor="white",
    handlelength=2.5,
    handleheight=1.5,
    columnspacing=0.5,
)

# Make legend title bold
legend.get_title().set_fontweight("bold")

# ============================================================
# ADD TITLE AND ANNOTATIONS
# ============================================================
ax.set_title(
    "ZEYBEK-3 Model: Idealized Geological Map\n"
    "Fault-Controlled Kimberlite Pipe in a Craton",
    fontsize=18,
    fontweight="bold",
    pad=25,
)

ax.text(
    x1,
    28.2,
    "Distinct Lithological Units (L1-L13) with Unique Colors",
    fontsize=12,
    ha="center",
    style="italic",
    color="#555555",
)

# ============================================================
# ADD RULE ANNOTATION
# ============================================================
rule_text = "CORE RULE:\n"
rule_text += "IF L1 (Epiclastics) is fault-bounded by F12 and F13\n"
rule_text += "THEN K is located at the centroid (x₁, y₁₄)\n"
rule_text += "where x₁ = (x₀ + x₂)/2 = (0+6)/2 = 3\n"
rule_text += "and y₁₄ = (y₁₃ + y₁₅)/2 = (13+15)/2 = 14"

ax.annotate(
    rule_text,
    xy=(k_x, k_y),
    xycoords="data",
    xytext=(x2 + 3.0, k_y + 4),
    fontsize=10,
    fontweight="bold",
    bbox=dict(
        boxstyle="round,pad=0.7",
        facecolor="yellow",
        alpha=0.85,
        edgecolor="orange",
        linewidth=2,
    ),
    arrowprops=dict(
        arrowstyle="->", connectionstyle="arc3,rad=0.2", color="red", linewidth=2.5
    ),
)

# ============================================================
# COORDINATE LABELS FOR KEY POINTS
# ============================================================
# L1 corner coordinates
ax.text(
    x0 - 0.3,
    y[13] - 0.3,
    f"({x0}, {y[13]})",
    fontsize=8,
    color="darkgreen",
    ha="right",
    va="top",
)
ax.text(
    x2 + 0.3,
    y[15] + 0.3,
    f"({x2}, {y[15]})",
    fontsize=8,
    color="darkgreen",
    ha="left",
    va="bottom",
)

# Kimberlite Pipe K coordinates
ax.text(
    k_x + 0.8,
    k_y + 0.3,
    f"K ({k_x}, {k_y})",
    fontsize=10,
    fontweight="bold",
    color="darkred",
    bbox=dict(boxstyle="round,pad=0.3", facecolor="white", alpha=0.8),
)

# ============================================================
# FINAL FORMATTING
# ============================================================
ax.set_xlabel(
    "X Coordinate (East / Horizontal Distance)", fontsize=13, fontweight="bold"
)
ax.set_ylabel(
    "Y Coordinate (North / Stratigraphic Depth / Elevation)",
    fontsize=13,
    fontweight="bold",
)

ax.set_xlim(-2.5, 8.5)
ax.set_ylim(-1.5, 28.5)

ax.set_xticks([])
ax.set_yticks([])

plt.tight_layout()

# ============================================================
# SAVE FIGURES
# ============================================================
plt.savefig(
    "ZEYBEK3_Figure1_With_Legend.png", dpi=300, bbox_inches="tight", facecolor="white"
)
plt.savefig("ZEYBEK3_Figure1_With_Legend.pdf", bbox_inches="tight", facecolor="white")

plt.show()

# ============================================================
# PRINT SUMMARY
# ============================================================
print("=" * 70)
print("ZEYBEK-3 MODEL FIGURE 1 - WITH COMPLETE LEGEND")
print("=" * 70)
print("✓ Comprehensive legend with all elements")
print("✓ Lithological Units (L1-L13) with distinct colors")
print("✓ Structural Elements (Faults, Compartment, Boundaries)")
print("✓ Target Element (Kimberlite Pipe K)")
print("✓ Coordinate System (X and Y axes)")
print("✓ Centroid point highlighted")
print("✓ Regular fault sequence F1-F24 displayed")
print("")
print("Legend Contents:")
print("  ■ L1: Epiclastics (Kimberlite Crater) ★")
print("  ■ L2-L13: All lithological units with colors")
print("  ─── Major Faults (F1, F12, F13, F24)")
print("  ─ ─ Intermediate Faults (F2-F11, F14-F23)")
print("  ░░░ Fault-Bounded Compartment")
print("  ● K: Kimberlite Pipe (Diamond) 💎")
print("  █ Centroid Point (x₁, y₁₄)")
print("")
print("Fault Sequence (Top to Bottom):")
print("  F1 (y=26) - MAJOR")
for i in range(2, 12):
    print(f"  F{i} (y={27 - i}) - intermediate")
print("  F12 (y=15) - MAJOR ★ Bounds L1")
print("  F13 (y=13) - MAJOR ★ Bounds L1")
for i in range(14, 24):
    print(f"  F{i} (y={27 - i}) - intermediate")
print("  F24 (y=1) - MAJOR")
print("")
print("Output Files:")
print("  - ZEYBEK3_Figure1_With_Legend.png")
print("  - ZEYBEK3_Figure1_With_Legend.pdf")
print("=" * 70)

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

"""
ZEYBEK-3 Model: Figure 2 - Workflow Diagram
Simple flowchart showing the step-by-step algorithm
"""

import matplotlib.pyplot as plt
import matplotlib.patches as patches
from matplotlib.patches import FancyBboxPatch, Circle, Rectangle, FancyArrowPatch
import numpy as np

# Set up the figure
fig, ax = plt.subplots(1, 1, figsize=(14, 18))
ax.set_xlim(-1, 11)
ax.set_ylim(-1, 22)
ax.set_aspect("equal")
ax.axis("off")

# ============================================================
# DEFINE STYLES
# ============================================================
# Colors
colors = {
    "start_end": "#4CAF50",  # Green for Start/End
    "process": "#2196F3",  # Blue for Process
    "decision": "#FF9800",  # Orange for Decision
    "data": "#9C27B0",  # Purple for Data Input
    "output": "#F44336",  # Red for Output
    "arrow": "#333333",  # Dark gray for arrows
    "text": "#FFFFFF",  # White text
    "box_bg": "#FFFFFF",  # White background for text
}


# ============================================================
# FUNCTION TO CREATE ROUNDED BOXES
# ============================================================
def draw_box(
    x,
    y,
    width,
    height,
    text,
    color,
    text_color="white",
    fontsize=10,
    fontweight="bold",
    alpha=1.0,
):
    """Draw a rounded rectangle with text"""
    box = FancyBboxPatch(
        (x - width / 2, y - height / 2),
        width,
        height,
        boxstyle="round,pad=0.15,rounding_size=0.15",
        facecolor=color,
        edgecolor="black",
        linewidth=2,
        alpha=alpha,
    )
    ax.add_patch(box)

    # Add text
    ax.text(
        x,
        y,
        text,
        ha="center",
        va="center",
        fontsize=fontsize,
        fontweight=fontweight,
        color=text_color,
        wrap=True,
    )
    return box


# ============================================================
# FUNCTION TO DRAW ARROWS
# ============================================================
def draw_arrow(
    x1,
    y1,
    x2,
    y2,
    color="#333333",
    linewidth=2,
    arrowstyle="->",
    connectionstyle="arc3,rad=0.0",
):
    """Draw an arrow between two points"""
    arrow = FancyArrowPatch(
        (x1, y1),
        (x2, y2),
        arrowstyle=arrowstyle,
        connectionstyle=connectionstyle,
        color=color,
        linewidth=linewidth,
        mutation_scale=20,
    )
    ax.add_patch(arrow)


# ============================================================
# DRAW THE WORKFLOW
# ============================================================

# Step 1: START
draw_box(5, 20.5, 3, 0.8, "START\nInitialize Targeting", colors["start_end"])

# Arrow to Step 2
draw_arrow(5, 20.1, 5, 19.2)

# Step 2: Define Target
draw_box(5, 18.5, 3.5, 0.8, "Define Target\nKimberlite Pipe (K)", colors["process"])

# Arrow to Data Input
draw_arrow(5, 18.1, 5, 17.2)

# Step 3: Data Input - Lithological Data
draw_box(5, 16.5, 3.5, 0.8, "Input Lithological Data\n(L1-L13)", colors["data"])

# Arrow to next data
draw_arrow(5, 16.1, 5, 15.2)

# Step 4: Data Input - Fault Data
draw_box(5, 14.5, 3.5, 0.8, "Input Fault Data\n(F1-F24)", colors["data"])

# Arrow to mapping
draw_arrow(5, 14.1, 5, 13.2)

# Step 5: Mapping
draw_box(
    5,
    12.5,
    4,
    0.8,
    "Map L1 Unit & Faults\n(Crater-Facies Epiclastics)",
    colors["process"],
)

# Arrow to decision
draw_arrow(5, 12.1, 5, 11.2)

# Step 6: Decision - Is L1 fault-bounded?
draw_box(5, 10.5, 4.5, 0.9, "Is L1 bounded by\nF12 and F13?", colors["decision"])

# Branch: YES (right arrow)
draw_arrow(7.25, 10.5, 8.5, 10.5, connectionstyle="arc3,rad=0.0")
ax.text(8.0, 10.8, "YES", fontsize=11, fontweight="bold", color="green")

# Branch: NO (left arrow)
draw_arrow(2.75, 10.5, 1.5, 10.5, connectionstyle="arc3,rad=0.0")
ax.text(2.0, 10.8, "NO", fontsize=11, fontweight="bold", color="red")

# NO branch: Stop/Return
draw_box(1.5, 9.0, 3, 0.7, "STOP\nConditions Not Met", colors["start_end"])
draw_arrow(1.5, 9.7, 1.5, 9.35)

# YES branch: Calculate Centroid
draw_box(8.5, 9.0, 3.5, 0.8, "Calculate Centroid\nof L1 Polygon", colors["process"])
draw_arrow(8.5, 9.7, 8.5, 9.35)

# Arrow to apply rule
draw_arrow(8.5, 8.6, 8.5, 7.7)

# Step 7: Apply Core Rule
draw_box(8.5, 7.0, 3.5, 0.8, "Apply Core Rule\nK = Centroid(L1)", colors["process"])

# Arrow to output
draw_arrow(8.5, 6.6, 8.5, 5.7)

# Step 8: Output
draw_box(8.5, 5.0, 3.5, 0.8, "Output Target\nCoordinate K(x₁, y₁₄)", colors["output"])

# Arrow to end
draw_arrow(8.5, 4.6, 8.5, 3.7)

# Step 9: END
draw_box(8.5, 3.0, 2.5, 0.7, "END", colors["start_end"])

# ============================================================
# ADD NUMBERED STEPS
# ============================================================
step_positions = [
    (0.5, 20.5),
    (0.5, 18.5),
    (0.5, 16.5),
    (0.5, 14.5),
    (0.5, 12.5),
    (0.5, 10.5),
    (6.0, 9.0),  # NO branch
    (6.0, 7.0),  # YES branch centroid
    (6.0, 5.0),  # Output
    (6.0, 3.0),  # End
]

# Add step numbers on the left side
for i, (x, y) in enumerate(step_positions[:6], 1):
    ax.text(
        x,
        y,
        f"Step {i}",
        fontsize=10,
        fontweight="bold",
        color="#555555",
        ha="center",
        va="center",
    )

# YES branch steps (steps 6b, 7, 8, 9)
step_labels = ["", "Step 6b", "Step 7", "Step 8", "Step 9"]
for i, (label, (x, y)) in enumerate(zip(step_labels, step_positions[6:]), 6):
    if label:
        ax.text(
            x,
            y,
            label,
            fontsize=10,
            fontweight="bold",
            color="#555555",
            ha="center",
            va="center",
        )

# ============================================================
# ADD LEGEND
# ============================================================
legend_y = 0.5
legend_x = 5

# Legend background
legend_box = FancyBboxPatch(
    (1, 0.9),
    9,
    1.2,
    boxstyle="round,pad=0.1,rounding_size=0.05",
    facecolor="white",
    edgecolor="black",
    linewidth=2,
    alpha=0.95,
)
ax.add_patch(legend_box)

# Legend title
ax.text(5.5, 2.0, "LEGEND", fontsize=12, fontweight="bold", ha="center", va="center")

# Legend items
legend_items = [
    ("Start/End", colors["start_end"]),
    ("Process", colors["process"]),
    ("Decision", colors["decision"]),
    ("Data Input", colors["data"]),
    ("Output", colors["output"]),
]

for i, (label, color) in enumerate(legend_items):
    x_pos = 1.5 + (i * 1.7)
    # Draw small colored box
    box = FancyBboxPatch(
        (x_pos - 0.4, 1.3),
        0.8,
        0.5,
        boxstyle="round,pad=0.05,rounding_size=0.05",
        facecolor=color,
        edgecolor="black",
        linewidth=1.5,
    )
    ax.add_patch(box)
    ax.text(x_pos, 0.9, label, fontsize=8, ha="center", va="center")

# ============================================================
# ADD TITLE
# ============================================================
ax.text(
    5,
    21.8,
    "ZEYBEK-3 Model: Workflow Diagram",
    fontsize=18,
    fontweight="bold",
    ha="center",
    va="center",
)
ax.text(
    5,
    21.3,
    "Step-by-Step Algorithm for Kimberlite Pipe Targeting",
    fontsize=12,
    ha="center",
    va="center",
    style="italic",
    color="#555555",
)

# ============================================================
# ADD FORMULA BOX (optional enhancement)
# ============================================================
formula_box = FancyBboxPatch(
    (0.5, 3.8),
    3.5,
    0.9,
    boxstyle="round,pad=0.1,rounding_size=0.1",
    facecolor="#FFF9C4",
    edgecolor="#F57F17",
    linewidth=2,
    alpha=0.9,
)
ax.add_patch(formula_box)
ax.text(
    2.25,
    4.45,
    "K = Centroid(L1)",
    fontsize=12,
    fontweight="bold",
    ha="center",
    va="center",
)
ax.text(
    2.25,
    4.15,
    "x₁ = (x₀ + x₂)/2",
    fontsize=9,
    ha="center",
    va="center",
    color="#555555",
)
ax.text(
    2.25,
    3.9,
    "y₁₄ = (y₁₃ + y₁₅)/2",
    fontsize=9,
    ha="center",
    va="center",
    color="#555555",
)

# Arrow from formula to centroid calculation
draw_arrow(
    4.0, 4.25, 6.0, 4.25, connectionstyle="arc3,rad=0.0", linewidth=1.5, color="#F57F17"
)

# ============================================================
# SAVE FIGURE
# ============================================================
plt.tight_layout()
plt.savefig(
    "ZEYBEK3_Figure2_Workflow.png", dpi=300, bbox_inches="tight", facecolor="white"
)
plt.savefig("ZEYBEK3_Figure2_Workflow.pdf", bbox_inches="tight", facecolor="white")

plt.show()

# ============================================================
# PRINT SUMMARY
# ============================================================
print("=" * 70)
print("ZEYBEK-3 MODEL: FIGURE 2 - WORKFLOW DIAGRAM")
print("=" * 70)
print("")
print("Workflow Steps:")
print("  Step 1: START - Initialize Targeting")
print("  Step 2: Define Target - Kimberlite Pipe (K)")
print("  Step 3: Input Lithological Data (L1-L13)")
print("  Step 4: Input Fault Data (F1-F24)")
print("  Step 5: Map L1 Unit & Faults")
print("  Step 6: Decision - Is L1 bounded by F12 and F13?")
print("     ├─ NO: STOP - Conditions Not Met")
print("     └─ YES: Continue")
print("  Step 7: Apply Core Rule - K = Centroid(L1)")
print("  Step 8: Output Target Coordinate K(x₁, y₁₄)")
print("  Step 9: END")
print("")
print("Output Files:")
print("  - ZEYBEK3_Figure2_Workflow.png")
print("  - ZEYBEK3_Figure2_Workflow.pdf")
print("=" * 70)

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import numpy as np
import pandas as pd

# Major kimberlite provinces with approximate coordinates (latitude, longitude)
# Data compiled from: Gittins (1996), Tappe et al. (2018), Kjarsgaard et al. (2019), Giuliani et al. (2023)
kimberlite_provinces = {
    "Africa": {
        "Kaapvaal Craton (South Africa)": (-28.0, 26.0),
        "Zimbabwe Craton": (-19.0, 30.0),
        "Tanzania Craton": (-5.0, 35.0),
        "West African Craton (Guinea)": (10.0, -10.0),
        "Angola Craton": (-9.0, 18.0),
        "Lesotho": (-29.5, 28.5),
        "Botswana (Orapa)": (-21.3, 25.3),
        "Botswana (Jwaneng)": (-24.5, 24.5),
        "Sierra Leone": (8.5, -12.0),
        "Liberia": (6.5, -9.0),
    },
    "North America": {
        "Slave Craton (Canada)": (64.0, -110.0),
        "Superior Craton (Canada)": (52.0, -85.0),
        "Wyoming Craton (USA)": (43.0, -107.0),
        "Colorado-Wyoming": (40.0, -105.0),
        "Arkansas (USA)": (34.5, -92.5),
        "Kirkland Lake (Canada)": (48.0, -80.0),
        "Saskatchewan (Canada)": (56.0, -104.0),
    },
    "South America": {
        "São Francisco Craton (Brazil)": (-15.0, -44.0),
        "Amazon Craton (Brazil)": (-5.0, -60.0),
        "Venezuela": (6.0, -66.0),
        "Guyana Shield": (4.0, -60.0),
    },
    "Russia/Asia": {
        "Siberian Craton (Russia)": (65.0, 112.0),
        "Arkhangelsk (Russia)": (65.0, 42.0),
        "Kola Peninsula (Russia)": (68.0, 35.0),
        "Aldan Shield (Russia)": (58.0, 126.0),
        "China (North China Craton)": (38.0, 115.0),
        "India (Dharwar Craton)": (15.0, 77.0),
        "India (Bundelkhand Craton)": (25.0, 80.0),
        "Korea": (38.5, 128.0),
    },
    "Australia": {
        "West Australian Craton": (-24.0, 128.0),
        "Kimberley Region": (-16.0, 126.0),
        "Ellendale (Australia)": (-17.5, 124.5),
        "Argyle (Australia)": (-16.7, 128.4),
    },
    "Europe": {
        "Finland (Karelian Craton)": (64.0, 28.0),
        "Ukraine (Ukrainian Shield)": (48.0, 32.0),
    },
}

# Create the map
fig = plt.figure(figsize=(16, 10))
ax = plt.axes(projection=ccrs.Robinson(central_longitude=0))

# Add map features
ax.add_feature(cfeature.LAND, facecolor="#f5f5dc", edgecolor="none", alpha=0.8)
ax.add_feature(cfeature.OCEAN, facecolor="#e8f4f8", alpha=0.7)
ax.add_feature(cfeature.COASTLINE, linewidth=0.5, alpha=0.6)
ax.add_feature(cfeature.BORDERS, linewidth=0.3, alpha=0.4, linestyle="--")

# Add gridlines
gl = ax.gridlines(
    draw_labels=True, linewidth=0.2, color="gray", alpha=0.3, linestyle="--"
)
gl.top_labels = False
gl.right_labels = False
gl.left_labels = True
gl.bottom_labels = True

# Define colors for each continent
continent_colors = {
    "Africa": "#e74c3c",  # Red
    "North America": "#3498db",  # Blue
    "South America": "#FFFF00",  # Yellow
    "Russia/Asia": "#f39c12",  # Orange
    "Australia": "#9b59b6",  # Purple
    "Europe": "#1abc9c",  # Turquoise
}

# Plot each kimberlite province
for continent, provinces in kimberlite_provinces.items():
    color = continent_colors.get(continent, "#7f8c8d")

    for name, (lat, lon) in provinces.items():
        # Plot marker
        ax.plot(
            lon,
            lat,
            marker="o",
            color=color,
            markersize=8,
            markeredgecolor="black",
            markeredgewidth=0.5,
            transform=ccrs.PlateCarree(),
            label=name if list(provinces.keys()).index(name) == 0 else "",
        )

        # Add small label for major occurrences
        if name in [
            "Slave Craton (Canada)",
            "Kaapvaal Craton (South Africa)",
            "Siberian Craton (Russia)",
            "West Australian Craton",
            "São Francisco Craton (Brazil)",
        ]:
            ax.text(
                lon - 20,
                lat + 3,
                name.split("(")[0].strip(),
                fontsize=8,
                transform=ccrs.PlateCarree(),
                alpha=0.7,
                bbox=dict(boxstyle="round,pad=0.2", facecolor="white", alpha=0.6),
            )

# Add legend (only show continent-level labels to avoid clutter)
from matplotlib.patches import Patch

legend_elements = [
    Patch(facecolor=color, edgecolor="black", label=continent)
    for continent, color in continent_colors.items()
]
ax.legend(
    handles=legend_elements,
    loc="lower left",
    title="Kimberlite Provinces by Continent",
    fontsize=10,
)

# Add title
plt.title(
    "Major Kimberlite Provinces and Occurrences Worldwide",
    fontsize=16,
    pad=20,
    weight="bold",
)

# Add subtitle
plt.text(
    0.5,
    -0.07,
    "Compiled from Gittins (1996), Tappe et al. (2018),\nKjarsgaard et al. (2019), Giuliani et al. (2023)",
    transform=plt.gca().transAxes,
    fontsize=9,
    ha="center",
    alpha=0.6,
)

# Add footnote for major clusters
plt.text(
    0.5,
    -0.10,
    "Major cratonic clusters: Slave, Superior, Kaapvaal, Zimbabwe, West African, Siberian, West Australian, São Francisco",
    transform=plt.gca().transAxes,
    fontsize=8,
    ha="center",
    alpha=0.5,
    style="italic",
)

# Save the map
plt.tight_layout()
plt.savefig("kimberlite_provinces_map.png", dpi=300, bbox_inches="tight")
plt.show()

# Print summary statistics
print("\n" + "=" * 60)
print("KIMBERLITE PROVINCE SUMMARY")
print("=" * 60)
print(
    f"Total provinces/occurrences plotted: {sum(len(v) for v in kimberlite_provinces.values())}"
)
print(f"Continents represented: {len(kimberlite_provinces)}")
print("\nCount by continent:")
for continent, provinces in kimberlite_provinces.items():
    print(f"  {continent}: {len(provinces)}")
print("=" * 60)

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX




































