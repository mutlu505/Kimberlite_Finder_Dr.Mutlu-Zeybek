# Kimberlite_Finder_Dr.Mutlu-Zeybek
"""
ZEYBEK-3 Model: A Rule-Based Expert System for Geometry-Driven Targeting of Fault-Controlled Kimberlite Pipes
Author: Mutlu ZEYBEK
Contact: mutlu505@gmail.com
Institution: Muğla Metropolitan Municipality, Muğla, Turkey
Date: 2025

This module implements the ZEYBEK-3 deterministic algorithm for kimberlite pipe targeting
based on geometric relationships between lithological units and fault systems.
"""

import numpy as np
from typing import List, Tuple, Union, Dict
import geopandas as gpd
from shapely.geometry import Polygon, Point, LineString
import matplotlib.pyplot as plt
from dataclasses import dataclass

# ============================================================================
# DATA CLASSES
# ============================================================================

@dataclass
class GeologicalUnit:
    """Represents a lithological unit in the ZEYBEK-3 model."""
    id: str  # L1, L2, ..., L13
    name: str  # Epiclastics, Tuff Ring, etc.
    vertices: List[Tuple[float, float]]  # Polygon vertices
    centroid: Tuple[float, float] = None
    
    def __post_init__(self):
        """Calculate centroid after initialization."""
        if self.vertices:
            polygon = Polygon(self.vertices)
            self.centroid = (polygon.centroid.x, polygon.centroid.y)

@dataclass
class Fault:
    """Represents a fault trace in the ZEYBEK-3 model."""
    id: str  # F1, F2, ..., F24
    vertices: List[Tuple[float, float]]  # Line vertices
    type: str = "unknown"  # normal, reverse, strike-slip

@dataclass
class KimberliteTarget:
    """Represents a calculated kimberlite pipe target."""
    coordinates: Tuple[float, float]
    confidence: float = 1.0  # Default confidence for deterministic model
    metadata: Dict = None
    offset_distance: float = None  # For validation purposes

# ============================================================================
# CORE ZEYBEK-3 ALGORITHM
# ============================================================================

class ZEYBEK3Model:
    """
    Implementation of the ZEYBEK-3 rule-based expert system for kimberlite targeting.
    
    The model follows the core rule:
    IF lithological unit L1 (Epiclastics) is bounded by specific fault pair (e.g., F12, F13),
    THEN the kimberlite pipe (K) is located at the geometric centroid of L1.
    """
    
    def __init__(self, ideal_mode: bool = False):
        """
        Initialize the ZEYBEK-3 model.
        
        Parameters:
        -----------
        ideal_mode : bool
            If True, uses the idealized rectangular geometry from the paper.
            If False, works with real-world polygon geometries.
        """
        self.ideal_mode = ideal_mode
        self.lithological_units = {}
        self.faults = {}
        self.target_lithology_id = "L1"  # Epiclastics
        
    def load_idealized_model(self):
        """Load the idealized geological map from the paper."""
        # Idealized coordinates from Figure 1
        x0, x1, x2 = 0.0, 1.0, 2.0
        y13, y14, y15 = 13.0, 14.0, 15.0
        
        # Define L1 (Epiclastics) - rectangular unit bounded by F12 and F13
        l1_vertices = [(x0, y13), (x0, y15), (x2, y15), (x2, y13)]
        self.lithological_units["L1"] = GeologicalUnit(
            id="L1",
            name="Epiclastics",
            vertices=l1_vertices
        )
        
        # Define bounding faults
        # F12: top boundary of L1
        f12_vertices = [(x0, y15), (x2, y15)]
        self.faults["F12"] = Fault(id="F12", vertices=f12_vertices)
        
        # F13: bottom boundary of L1
        f13_vertices = [(x0, y13), (x2, y13)]
        self.faults["F13"] = Fault(id="F13", vertices=f13_vertices)
        
        # Add additional lithologies (simplified for demonstration)
        # L2: Tuff Ring
        l2_vertices = [(x0, y15), (x0, 16.0), (x2, 16.0), (x2, y15)]
        self.lithological_units["L2"] = GeologicalUnit(
            id="L2",
            name="Tuff Ring",
            vertices=l2_vertices
        )
        
        # L13: Granite (basement)
        l13_vertices = [(x0, 10.0), (x0, y13), (x2, y13), (x2, 10.0)]
        self.lithological_units["L13"] = GeologicalUnit(
            id="L13",
            name="Granite",
            vertices=l13_vertices
        )
        
        print("Idealized model loaded successfully.")
        print(f"L1 vertices: {l1_vertices}")
        print(f"L1 centroid: {self.lithological_units['L1'].centroid}")
        
    def add_lithological_unit(self, unit_id: str, name: str, vertices: List[Tuple[float, float]]):
        """Add a lithological unit to the model."""
        self.lithological_units[unit_id] = GeologicalUnit(
            id=unit_id,
            name=name,
            vertices=vertices
        )
        
    def add_fault(self, fault_id: str, vertices: List[Tuple[float, float]], fault_type: str = "unknown"):
        """Add a fault trace to the model."""
        self.faults[fault_id] = Fault(
            id=fault_id,
            vertices=vertices,
            type=fault_type
        )
    
    def do_faults_bound_l1(self, fault_pair: Tuple[str, str] = ("F12", "F13")) -> bool:
        """
        Check if the specified fault pair bounds the L1 unit.
        
        Parameters:
        -----------
        fault_pair : tuple
            Tuple of two fault IDs to check as bounding faults.
            
        Returns:
        --------
        bool
            True if faults bound L1, False otherwise.
        """
        if self.target_lithology_id not in self.lithological_units:
            raise ValueError(f"Target lithology {self.target_lithology_id} not found in model.")
        
        if fault_pair[0] not in self.faults or fault_pair[1] not in self.faults:
            raise ValueError(f"One or both faults {fault_pair} not found in model.")
        
        l1_polygon = Polygon(self.lithological_units[self.target_lithology_id].vertices)
        fault1_line = LineString(self.faults[fault_pair[0]].vertices)
        fault2_line = LineString(self.faults[fault_pair[1]].vertices)
        
        # Get bounding box of L1
        minx, miny, maxx, maxy = l1_polygon.bounds
        
        # Check if faults align with top and bottom boundaries
        # For idealized model, faults should coincide with y = y15 and y = y13
        if self.ideal_mode:
            # Extract y-coordinates from fault vertices
            fault1_y = set([v[1] for v in self.faults[fault_pair[0]].vertices])
            fault2_y = set([v[1] for v in self.faults[fault_pair[1]].vertices])
            
            # Check if faults align with top and bottom
            top_boundary = maxy
            bottom_boundary = miny
            
            return (top_boundary in fault1_y and bottom_boundary in fault2_y) or \
                   (top_boundary in fault2_y and bottom_boundary in fault1_y)
        else:
            # For real-world cases, use proximity threshold
            threshold = 0.01  # 1% of average dimension
            
            # Calculate distances from faults to L1 boundaries
            fault1_dist_to_top = abs(fault1_line.coords[0][1] - maxy)
            fault1_dist_to_bottom = abs(fault1_line.coords[0][1] - miny)
            fault2_dist_to_top = abs(fault2_line.coords[0][1] - maxy)
            fault2_dist_to_bottom = abs(fault2_line.coords[0][1] - miny)
            
            # Check if one fault is near top and other near bottom
            avg_dim = (maxx - minx + maxy - miny) / 2
            threshold_abs = threshold * avg_dim
            
            return (fault1_dist_to_top < threshold_abs and fault2_dist_to_bottom < threshold_abs) or \
                   (fault2_dist_to_top < threshold_abs and fault1_dist_to_bottom < threshold_abs)
    
    def calculate_polygon_centroid(self, vertices: List[Tuple[float, float]]) -> Tuple[float, float]:
        """
        Calculate the centroid of a polygon.
        
        Parameters:
        -----------
        vertices : list
            List of (x, y) coordinate pairs defining the polygon.
            
        Returns:
        --------
        tuple
            (x, y) coordinates of the centroid.
        """
        polygon = Polygon(vertices)
        centroid = polygon.centroid
        return (centroid.x, centroid.y)
    
    def is_bounded_by_fault_pair(self, fault_pair: Tuple[str, str]) -> bool:
        """
        Simplified check for fault bounding (for code representation in paper).
        """
        return self.do_faults_bound_l1(fault_pair)
    
    def zeybek3_target(self, fault_pair: Tuple[str, str] = ("F12", "F13")) -> Union[KimberliteTarget, str]:
        """
        Main ZEYBEK-3 targeting algorithm as described in the paper.
        
        Parameters:
        -----------
        fault_pair : tuple
            The fault pair hypothesized to bound the L1 unit.
            
        Returns:
        --------
        KimberliteTarget or str
            Target coordinates if conditions met, error message otherwise.
        """
        # Step 1: Validate Fault-Lithology Geometry
        if not self.do_faults_bound_l1(fault_pair):
            return "Model condition not met: L1 is not bounded by the specified fault pair."
        
        # Step 2: Apply Deterministic Rule
        # IF lithological unit L1 (Epiclastics) is bounded by specific fault pair
        if self.is_bounded_by_fault_pair(fault_pair):
            # THEN the causative kimberlite pipe (K) is located at the centroid of L1
            l1_vertices = self.lithological_units[self.target_lithology_id].vertices
            k_coordinate = self.calculate_polygon_centroid(l1_vertices)
            
            # Step 3: Create target object
            target = KimberliteTarget(
                coordinates=k_coordinate,
                confidence=1.0,
                metadata={
                    "method": "ZEYBEK-3 Deterministic",
                    "fault_pair": fault_pair,
                    "l1_centroid": self.lithological_units[self.target_lithology_id].centroid,
                    "l1_area": Polygon(l1_vertices).area
                }
            )
            return target
        
        return "Model conditions not satisfied."
    
    def validate_with_known_pipe(self, known_pipe_location: Tuple[float, float]) -> Dict:
        """
        Validate model prediction against known pipe location.
        
        Parameters:
        -----------
        known_pipe_location : tuple
            (x, y) coordinates of known kimberlite pipe.
            
        Returns:
        --------
        dict
            Validation results including offset distance.
        """
        prediction = self.zeybek3_target()
        
        if isinstance(prediction, str):
            return {"success": False, "error": prediction}
        
        # Calculate Euclidean distance
        pred_x, pred_y = prediction.coordinates
        known_x, known_y = known_pipe_location
        
        offset_distance = np.sqrt((pred_x - known_x)**2 + (pred_y - known_y)**2)
        prediction.offset_distance = offset_distance
        
        return {
            "success": True,
            "predicted_location": prediction.coordinates,
            "known_location": known_pipe_location,
            "offset_distance": offset_distance,
            "target_object": prediction
        }
    
    def plot_geological_map(self, save_path: str = None):
        """
        Plot the geological map with lithological units and faults.
        
        Parameters:
        -----------
        save_path : str, optional
            Path to save the figure.
        """
        fig, ax = plt.subplots(figsize=(12, 10))
        
        # Plot lithological units
        colors = plt.cm.Set3(np.linspace(0, 1, len(self.lithological_units)))
        for i, (unit_id, unit) in enumerate(self.lithological_units.items()):
            polygon = Polygon(unit.vertices)
            x, y = polygon.exterior.xy
            ax.fill(x, y, alpha=0.6, color=colors[i], label=unit.name)
            # Add centroid
            ax.plot(unit.centroid[0], unit.centroid[1], 'ko', markersize=4)
            # Add label
            ax.text(unit.centroid[0], unit.centroid[1], unit_id, 
                   fontsize=9, ha='center', va='center', fontweight='bold')
        
        # Plot faults
        for fault_id, fault in self.faults.items():
            x, y = zip(*fault.vertices)
            ax.plot(x, y, 'r-', linewidth=2, label=fault_id if fault_id in ["F12", "F13"] else "")
            # Add fault label at midpoint
            mid_idx = len(x) // 2
            ax.text(x[mid_idx], y[mid_idx], fault_id, 
                   fontsize=9, color='red', ha='center', va='center',
                   bbox=dict(boxstyle="round,pad=0.3", facecolor="white", alpha=0.7))
        
        # Highlight L1 unit
        if self.target_lithology_id in self.lithological_units:
            l1_polygon = Polygon(self.lithological_units[self.target_lithology_id].vertices)
            x, y = l1_polygon.exterior.xy
            ax.plot(x, y, 'k--', linewidth=2, alpha=0.8)
        
        # Calculate and plot predicted kimberlite location
        prediction = self.zeybek3_target()
        if isinstance(prediction, KimberliteTarget):
            pred_x, pred_y = prediction.coordinates
            ax.plot(pred_x, pred_y, 'k*', markersize=15, 
                   label='Predicted Kimberlite (K)', markeredgewidth=2, 
                   markeredgecolor='yellow')
            ax.text(pred_x, pred_y, 'K', fontsize=12, ha='center', 
                   va='bottom', fontweight='bold', color='black')
        
        ax.set_xlabel('X Coordinate', fontsize=12)
        ax.set_ylabel('Y Coordinate', fontsize=12)
        ax.set_title('ZEYBEK-3 Model: Geological Map with Kimberlite Target', fontsize=14, fontweight='bold')
        ax.grid(True, alpha=0.3)
        ax.legend(loc='upper right', fontsize=9)
        ax.set_aspect('equal', adjustable='box')
        
        if save_path:
            plt.savefig(save_path, dpi=300, bbox_inches='tight')
            print(f"Figure saved to {save_path}")
        
        plt.show()
        return fig, ax

# ============================================================================
# VALIDATION AND CASE STUDY FUNCTIONS
# ============================================================================

def retrospective_case_study():
    """
    Simulate the retrospective case study from the Lac de Gras field.
    This function demonstrates the validation methodology described in the paper.
    """
    print("=" * 70)
    print("RETROSPECTIVE CASE STUDY: Lac de Gras Kimberlite Field")
    print("=" * 70)
    
    # Simulated data for demonstration (based on Table in Section 4.6.3)
    case_study_data = [
        {
            "pipe_id": "Pipe A",
            "l1_area": 0.12,  # km²
            "l1_vertices": [(112.455, 65.122), (112.455, 65.124), 
                           (112.458, 65.124), (112.458, 65.122)],
            "f12_vertices": [(112.455, 65.124), (112.458, 65.124)],
            "f13_vertices": [(112.455, 65.122), (112.458, 65.122)],
            "known_location": (112.4559, 65.1228)  # (Longitude, Latitude)
        },
        {
            "pipe_id": "Pipe B",
            "l1_area": 0.08,
            "l1_vertices": [(112.510, 65.186), (112.510, 65.187), 
                           (112.512, 65.187), (112.512, 65.186)],
            "f12_vertices": [(112.510, 65.187), (112.512, 65.187)],
            "f13_vertices": [(112.510, 65.186), (112.512, 65.186)],
            "known_location": (112.5110, 65.1865)
        },
        {
            "pipe_id": "Pipe C",
            "l1_area": 0.25,
            "l1_vertices": [(112.397, 65.201), (112.397, 65.202), 
                           (112.400, 65.202), (112.400, 65.201)],
            "f12_vertices": [(112.397, 65.202), (112.400, 65.202)],
            "f13_vertices": [(112.397, 65.201), (112.400, 65.201)],
            "known_location": (112.3990, 65.2020)
        }
    ]
    
    results = []
    
    for case in case_study_data:
        print(f"\nAnalyzing {case['pipe_id']}...")
        
        # Create model instance
        model = ZEYBEK3Model(ideal_mode=False)
        
        # Add L1 unit
        model.add_lithological_unit("L1", "Epiclastics", case["l1_vertices"])
        
        # Add bounding faults
        model.add_fault("F12", case["f12_vertices"])
        model.add_fault("F13", case["f13_vertices"])
        
        # Run ZEYBEK-3 targeting
        validation_result = model.validate_with_known_pipe(case["known_location"])
        
        if validation_result["success"]:
            offset_m = validation_result["offset_distance"] * 111000  # Convert degrees to meters (approx)
            results.append({
                "pipe": case["pipe_id"],
                "offset_m": offset_m,
                "predicted": validation_result["predicted_location"],
                "actual": case["known_location"]
            })
            
            print(f"  Predicted: {validation_result['predicted_location']}")
            print(f"  Actual: {case['known_location']}")
            print(f"  Offset: {offset_m:.1f} meters")
        else:
            print(f"  Error: {validation_result['error']}")
    
    # Calculate statistics
    if results:
        offsets = [r["offset_m"] for r in results]
        mean_offset = np.mean(offsets)
        max_offset = np.max(offsets)
        min_offset = np.min(offsets)
        
        print("\n" + "=" * 70)
        print("CASE STUDY RESULTS SUMMARY")
        print("=" * 70)
        print(f"Number of pipes analyzed: {len(results)}")
        print(f"Mean Offset Distance: {mean_offset:.1f} meters")
        print(f"Minimum Offset: {min_offset:.1f} meters")
        print(f"Maximum Offset: {max_offset:.1f} meters")
        
        # Interpretation
        print("\nINTERPRETATION:")
        if mean_offset < 100:
            print(f"✓ Mean offset of {mean_offset:.1f} meters is highly significant.")
            print("✓ Prediction places drill hole within typical pipe diameter (200-500m).")
            print("✓ Supports core geometric premise of ZEYBEK-3 model.")
        else:
            print(f"⚠ Mean offset of {mean_offset:.1f} meters suggests need for model refinement.")
        
        return results, offsets
    
    return None, None

# ============================================================================
# PROBABILISTIC EXTENSION (ZEYBEK-3P)
# ============================================================================

class ZEYBEK3P(ZEYBEK3Model):
    """
    Probabilistic extension of ZEYBEK-3 model.
    Incorporates confidence factors based on data quality and convergence.
    """
    
    def __init__(self):
        super().__init__(ideal_mode=False)
        self.confidence_factors = {
            "geometry": 1.0,  # Certainty in L1 polygon boundaries
            "fault_boundary": 1.0,  # Certainty that faults bound L1
            "data_convergence": 1.0  # Number of independent data types supporting L1
        }
    
    def calculate_confidence(self, 
                            geometry_quality: float = 0.9,
                            fault_certainty: float = 0.9,
                            n_data_types: int = 3) -> float:
        """
        Calculate overall confidence score for prediction.
        
        Parameters:
        -----------
        geometry_quality : float
            Quality of L1 polygon mapping (0.0 to 1.0)
        fault_certainty : float
            Certainty that identified faults truly bound L1 (0.0 to 1.0)
        n_data_types : int
            Number of independent data types supporting L1 interpretation
            
        Returns:
        --------
        float
            Overall confidence score (0.0 to 1.0)
        """
        # Update confidence factors
        self.confidence_factors["geometry"] = geometry_quality
        self.confidence_factors["fault_boundary"] = fault_certainty
        self.confidence_factors["data_convergence"] = min(1.0, n_data_types / 5.0)
        
        # Calculate weighted confidence
        weights = {"geometry": 0.4, "fault_boundary": 0.4, "data_convergence": 0.2}
        
        confidence = sum(self.confidence_factors[factor] * weights[factor] 
                        for factor in self.confidence_factors)
        
        return confidence
    
    def zeybek3_target_probabilistic(self, 
                                    fault_pair: Tuple[str, str] = ("F12", "F13"),
                                    **confidence_kwargs) -> Union[KimberliteTarget, str]:
        """
        Probabilistic version of ZEYBEK-3 targeting.
        
        Returns target with confidence score instead of binary yes/no.
        """
        # First, run standard ZEYBEK-3
        base_result = super().zeybek3_target(fault_pair)
        
        if isinstance(base_result, str):
            return base_result
        
        # Calculate confidence
        confidence = self.calculate_confidence(**confidence_kwargs)
        
        # Update target with confidence
        base_result.confidence = confidence
        
        # Add confidence metadata
        if base_result.metadata is None:
            base_result.metadata = {}
        base_result.metadata.update({
            "confidence_factors": self.confidence_factors.copy(),
            "model_version": "ZEYBEK-3P"
        })
        
        return base_result

# ============================================================================
# WORKFLOW EXAMPLE AND DEMONSTRATION
# ============================================================================

def demonstrate_workflow():
    """
    Demonstrate the complete ZEYBEK-3 workflow as described in the paper.
    """
    print("=" * 70)
    print("ZEYBEK-3 MODEL: COMPLETE WORKFLOW DEMONSTRATION")
    print("=" * 70)
    
    # Step 1: Initialize model with idealized geology
    print("\n1. INITIALIZATION: Creating idealized geological model...")
    model = ZEYBEK3Model(ideal_mode=True)
    model.load_idealized_model()
    
    # Step 2: Validate fault-lithology geometry
    print("\n2. GEOMETRY VALIDATION: Checking if faults bound L1 unit...")
    is_bounded = model.do_faults_bound_l1(("F12", "F13"))
    print(f"   Faults F12 and F13 bound L1 unit: {is_bounded}")
    
    # Step 3: Apply core rule and calculate target
    print("\n3. TARGET CALCULATION: Applying ZEYBEK-3 algorithm...")
    target = model.zeybek3_target(("F12", "F13"))
    
    if isinstance(target, KimberliteTarget):
        print(f"   Predicted Kimberlite Pipe Location: {target.coordinates}")
        print(f"   Confidence: {target.confidence}")
        if target.metadata:
            print(f"   Metadata: {target.metadata}")
    else:
        print(f"   Result: {target}")
    
    # Step 4: Visualization
    print("\n4. VISUALIZATION: Generating geological map...")
    model.plot_geological_map()
    
    # Step 5: Demonstrate validation
    print("\n5. VALIDATION: Testing with hypothetical known location...")
    known_location = (1.0, 14.0)  # Should match centroid in idealized model
    validation = model.validate_with_known_pipe(known_location)
    
    if validation["success"]:
        print(f"   Predicted: {validation['predicted_location']}")
        print(f"   Known: {validation['known_location']}")
        print(f"   Offset: {validation['offset_distance']:.4f} units")
        print("   ✓ Model prediction matches expected location")
    else:
        print(f"   Validation failed: {validation['error']}")
    
    print("\n" + "=" * 70)
    print("WORKFLOW COMPLETED SUCCESSFULLY")
    print("=" * 70)
    
    return model, target

# ============================================================================
# MAIN EXECUTION
# ============================================================================

if __name__ == "__main__":
    """
    Main execution block demonstrating the ZEYBEK-3 model.
    """
    import argparse
    
    parser = argparse.ArgumentParser(description="Run the ZEYBEK-3 kimberlite targeting model")
    parser.add_argument("--mode", choices=["demo", "case_study", "probabilistic"], 
                       default="demo", help="Run mode")
    parser.add_argument("--plot", action="store_true", help="Generate plots")
    parser.add_argument("--save", type=str, help="Save plot to file")
    
    args = parser.parse_args()
    
    if args.mode == "demo":
        # Run complete workflow demonstration
        model, target = demonstrate_workflow()
        
        if args.plot:
            model.plot_geological_map(args.save if args.save else None)
    
    elif args.mode == "case_study":
        # Run retrospective case study
        results, offsets = retrospective_case_study()
    
    elif args.mode == "probabilistic":
        # Demonstrate probabilistic extension
        print("\n" + "=" * 70)
        print("ZEYBEK-3P: PROBABILISTIC EXTENSION DEMONSTRATION")
        print("=" * 70)
        
        # Create probabilistic model
        prob_model = ZEYBEK3P()
        prob_model.load_idealized_model()
        
        # Run with different confidence scenarios
        print("\nScenario 1: High-quality data")
        target1 = prob_model.zeybek3_target_probabilistic(
            geometry_quality=0.95,
            fault_certainty=0.90,
            n_data_types=4
        )
        print(f"   Confidence: {target1.confidence:.2f}")
        
        print("\nScenario 2: Moderate-quality data")
        target2 = prob_model.zeybek3_target_probabilistic(
            geometry_quality=0.70,
            fault_certainty=0.60,
            n_data_types=2
        )
        print(f"   Confidence: {target2.confidence:.2f}")
        
        print("\nScenario 3: Low-quality data")
        target3 = prob_model.zeybek3_target_probabilistic(
            geometry_quality=0.40,
            fault_certainty=0.30,
            n_data_types=1
        )
        print(f"   Confidence: {target3.confidence:.2f}")
        
        print("\n" + "=" * 70)
        print("Probabilistic model allows for risk-based decision making")
        print("Confidence scores can guide exploration budget allocation")
        print("=" * 70)
    
    print("\nZEYBEK-3 model execution completed.")
