# Movement Project: Detailed Gap Analysis

**Analysis Date:** February 2026  
**Repository:** AnandMayank/movement (fork of neuroinformatics-unit/movement)  
**Current Version:** v0.1+

---

## Overview

This document provides a **detailed technical analysis** of gaps in the current codebase, specific to files and functions. Each gap includes:
- 📍 **Location:** Specific file paths and functions
- 🎯 **Impact:** Severity and user impact
- 🔧 **Solution:** Concrete implementation suggestions
- 📊 **Complexity:** Estimated effort and dependencies

---

## 1. Critical Gaps (High Impact, High Priority)

### Gap 1.1: Missing Outlier Detection in Filtering Module
**Location:** `movement/filtering.py`  
**Current State:** Only confidence-based filtering exists  
**Missing:** Automatic statistical outlier detection

**Impact:** 🔴 High
- Users must manually identify outliers in noisy data
- No standardized method for detecting tracking artifacts
- Critical for data quality assurance

**Proposed Solution:**
```python
# Add to movement/filtering.py

def detect_outliers(
    data: xr.DataArray,
    method: str = "iqr",  # 'iqr', 'zscore', 'isolation_forest'
    threshold: float = 1.5,
    window: int | None = None,
) -> xr.DataArray:
    """
    Detect outliers in pose tracking data.
    
    Parameters
    ----------
    data : xr.DataArray
        Input data with time dimension
    method : str
        Detection method:
        - 'iqr': Interquartile range (default)
        - 'zscore': Z-score based
        - 'isolation_forest': ML-based (requires sklearn)
    threshold : float
        Sensitivity parameter (method-dependent)
    window : int, optional
        Rolling window for local outlier detection
        
    Returns
    -------
    xr.DataArray
        Boolean mask where True indicates outliers
    """
    pass  # Implementation needed
```

**Files to Modify:**
- `movement/filtering.py` - Add new function
- `tests/test_unit/test_filtering.py` - Add tests
- `docs/source/api_index.html` - Add to API docs

**Dependencies:** `scipy.stats`, `sklearn` (optional)  
**Complexity:** Medium (8-16 hours)

---

### Gap 1.2: No HMM for Behavior State Detection
**Location:** New module needed: `movement/analysis/hmm.py`  
**Current State:** No state-based behavior analysis  
**Missing:** Hidden Markov Model integration

**Impact:** 🔴 High
- Cannot automatically segment behaviors into discrete states
- No temporal modeling of behavior sequences
- Limited use for ethology research

**Proposed Solution:**
```python
# New file: movement/analysis/hmm.py

import numpy as np
from hmmlearn import hmm
import xarray as xr


class BehaviorHMM:
    """Hidden Markov Model for behavior state detection."""
    
    def __init__(self, n_states: int = 3, **hmm_kwargs):
        """
        Initialize HMM model.
        
        Parameters
        ----------
        n_states : int
            Number of hidden states (behaviors)
        **hmm_kwargs
            Additional arguments for hmmlearn.GaussianHMM
        """
        self.model = hmm.GaussianHMM(n_components=n_states, **hmm_kwargs)
        self.n_states = n_states
        
    def fit(self, features: xr.Dataset) -> "BehaviorHMM":
        """Fit HMM to kinematic features."""
        pass
        
    def predict(self, features: xr.Dataset) -> xr.DataArray:
        """Predict behavior states for new data."""
        pass
        
    def get_transition_matrix(self) -> np.ndarray:
        """Get state transition probabilities."""
        return self.model.transmat_
```

**Files to Create:**
- `movement/analysis/` (new directory)
- `movement/analysis/__init__.py`
- `movement/analysis/hmm.py`
- `tests/test_unit/test_analysis/test_hmm.py`
- `examples/hmm_behavior_classification.py`

**Dependencies:** `hmmlearn`, `scikit-learn`  
**Complexity:** High (24-40 hours)

---

### Gap 1.3: Limited 3D Visualization
**Location:** `movement/plots/trajectory.py`  
**Current State:** Basic 3D trajectory plot exists  
**Missing:** Interactive 3D viewer with rich features

**Impact:** 🟡 Medium-High
- 3D pose data cannot be explored interactively
- No comparison between multiple individuals
- Limited publication-quality 3D figures

**Proposed Solution:**
```python
# Add to movement/plots/trajectory.py or new trajectory_3d.py

def plot_trajectory_3d_interactive(
    data: xr.Dataset,
    keypoint: str,
    individual: str | None = None,
    color_by: str = "time",  # 'time', 'speed', 'state'
    show_markers: bool = True,
    show_path: bool = True,
    backend: str = "plotly",  # 'plotly', 'pyvista', 'mayavi'
) -> go.Figure:  # or pyvista.Plotter
    """
    Create interactive 3D trajectory visualization.
    
    Features:
    - Rotation, zoom, pan controls
    - Animation along time axis
    - Multiple individuals overlay
    - Export to HTML/GIF
    """
    pass
```

**Files to Modify:**
- `movement/plots/trajectory.py` - Extend or create trajectory_3d.py
- `tests/test_unit/test_plots/test_trajectory_3d.py`
- Add optional dependency: `plotly` or `pyvista`

**Dependencies:** `plotly>=5.0` (lightweight) or `pyvista` (advanced)  
**Complexity:** Medium (12-20 hours)

---

## 2. Important Gaps (Medium Impact, Medium Priority)

### Gap 2.1: No Kalman Filtering
**Location:** `movement/filtering.py`  
**Current State:** Only basic filters (median, savgol, linear interpolation)  
**Missing:** State-space filtering with uncertainty estimates

**Impact:** 🟡 Medium
- Cannot leverage physics-based smoothing
- No prediction of occluded keypoints
- Suboptimal for noisy/missing data

**Code Snippet:**
```python
# Add to movement/filtering.py

from filterpy.kalman import KalmanFilter

def kalman_filter(
    data: xr.DataArray,
    process_noise: float = 0.01,
    measurement_noise: float = 0.1,
    initial_state: np.ndarray | None = None,
) -> xr.DataArray:
    """
    Apply Kalman filter to pose data.
    
    Assumes constant velocity motion model.
    """
    # Implementation using filterpy
    pass
```

**Dependencies:** `filterpy`  
**Complexity:** Medium (10-16 hours)

---

### Gap 2.2: Missing Batch Processing Utilities
**Location:** New file needed: `movement/utils/batch.py`  
**Current State:** No built-in multi-file processing  
**Missing:** Utilities for processing datasets in batch

**Impact:** 🟡 Medium
- Users must write custom loops for multiple files
- No parallelization support
- Difficult to process large experiments

**Code Snippet:**
```python
# New file: movement/utils/batch.py

from typing import Callable, List
from pathlib import Path
import multiprocessing as mp
from tqdm import tqdm

def process_files_parallel(
    file_paths: List[Path],
    processing_func: Callable,
    n_jobs: int = -1,
    show_progress: bool = True,
    **func_kwargs,
) -> List:
    """
    Process multiple files in parallel.
    
    Parameters
    ----------
    file_paths : list of Path
        Input files to process
    processing_func : callable
        Function to apply to each file
    n_jobs : int
        Number of parallel jobs (-1 = all CPUs)
    """
    pass
```

**Complexity:** Low-Medium (6-10 hours)

---

### Gap 2.3: No Circular/Elliptical ROIs
**Location:** `movement/roi/`  
**Current State:** Only Line and Polygon ROIs  
**Missing:** Circular and elliptical regions

**Impact:** 🟡 Medium
- Cannot represent circular arenas easily
- No support for elliptical zones (common in experiments)

**Code Snippet:**
```python
# New file: movement/roi/circle.py

from movement.roi.base import ROIBase
from shapely.geometry import Point
import numpy as np

class CircularROI(ROIBase):
    """Circular region of interest."""
    
    def __init__(self, center: tuple[float, float], radius: float):
        """
        Initialize circular ROI.
        
        Parameters
        ----------
        center : tuple of float
            (x, y) coordinates of center
        radius : float
            Radius in spatial units
        """
        self.center = np.array(center)
        self.radius = radius
        self._polygon = Point(center).buffer(radius)
        
    def contains_points(self, points: xr.DataArray) -> xr.DataArray:
        """Check if points are inside circle."""
        distances = np.linalg.norm(points - self.center, axis=-1)
        return distances <= self.radius
```

**Files to Create:**
- `movement/roi/circle.py`
- `movement/roi/ellipse.py`
- `tests/test_unit/test_roi/test_circle.py`
- `tests/test_unit/test_roi/test_ellipse.py`

**Complexity:** Low (4-8 hours each)

---

### Gap 2.4: Limited Error Recovery in I/O
**Location:** `movement/io/load_poses.py`, `movement/io/save_poses.py`  
**Current State:** Basic try-except, but limited recovery  
**Missing:** Graceful error handling and user guidance

**Impact:** 🟡 Medium
- Cryptic error messages confuse users
- File corruption causes crashes
- No suggestions for fixing issues

**Improvement Areas:**
```python
# In movement/io/load_poses.py

# Current:
try:
    ds = load_from_dlc(file_path)
except Exception as e:
    logger.error(f"Failed to load file: {e}")
    raise

# Improved:
try:
    ds = load_from_dlc(file_path)
except KeyError as e:
    raise ValueError(
        f"Missing required column in DLC file: {e}. "
        f"Expected columns: {REQUIRED_DLC_COLUMNS}. "
        f"Hint: Check if file was exported correctly from DLC."
    ) from e
except pd.errors.EmptyDataError:
    raise ValueError(
        f"File appears to be empty: {file_path}. "
        f"Hint: Check if pose estimation completed successfully."
    )
```

**Files to Modify:**
- `movement/io/load_poses.py`
- `movement/io/save_poses.py`
- `movement/io/load_bboxes.py`
- `movement/validators/files.py`

**Complexity:** Low-Medium (8-12 hours total)

---

## 3. Enhancement Opportunities (Lower Priority)

### Gap 3.1: No Automatic Report Generation
**Location:** New file: `movement/reports/`  
**Current State:** Users must create their own summaries  
**Missing:** Automated report generation (PDF/HTML)

**Impact:** 🟢 Low-Medium
- No standardized analysis summaries
- Difficult to share results with collaborators
- No publication-ready figures

**Proposed Features:**
- Automatic data quality report (missing values, confidence distribution)
- Kinematic summary statistics (mean velocity, path length, etc.)
- Trajectory visualization gallery
- Export to PDF (using reportlab) or HTML (using jinja2)

**Complexity:** High (20-30 hours)

---

### Gap 3.2: Missing Gait Analysis Module
**Location:** New directory: `movement/analysis/gait/`  
**Current State:** No specialized gait analysis tools  
**Missing:** Stride detection, symmetry analysis, phase relationships

**Impact:** 🟢 Low (niche use case)
- Useful for biomechanics and veterinary research
- Not aligned with core neuroscience focus
- Could be plugin or separate package

**Suggested Features:**
```python
# movement/analysis/gait.py

def detect_strides(
    limb_position: xr.DataArray,
    height_threshold: float = 0.01,
    min_duration: int = 5,
) -> List[Tuple[int, int]]:
    """Detect stride cycles from limb vertical position."""
    pass

def compute_stride_frequency(strides: List[Tuple[int, int]], fps: float) -> float:
    """Calculate stride frequency in Hz."""
    pass

def compute_limb_phase_relationship(
    limb1: xr.DataArray,
    limb2: xr.DataArray,
) -> float:
    """Compute phase difference between limbs."""
    pass
```

**Complexity:** High (30+ hours)

---

### Gap 3.3: Limited Type Hints in Older Code
**Location:** `movement/filtering.py`, `movement/transforms.py`  
**Current State:** Some functions lack type annotations  
**Missing:** Full type coverage for mypy compliance

**Impact:** 🟢 Low
- Reduces IDE autocomplete quality
- Harder to catch bugs during development
- Not critical for functionality

**Example Fix:**
```python
# Before:
def interpolate_over_time(data, max_gap=None, method="linear"):
    ...

# After:
from typing import Literal

def interpolate_over_time(
    data: xr.DataArray,
    max_gap: int | None = None,
    method: Literal["linear", "polynomial", "spline"] = "linear",
) -> xr.DataArray:
    ...
```

**Complexity:** Low (4-8 hours)

---

## 4. Testing Gaps

### Gap 4.1: Missing Edge Case Tests
**Location:** `tests/test_unit/test_filtering.py`, `tests/test_unit/test_kinematics/`  
**Current State:** Good coverage for normal cases  
**Missing:** Tests for edge cases

**Examples of Missing Tests:**
```python
# tests/test_unit/test_filtering.py

def test_filter_empty_dataset():
    """Test filtering with no data points."""
    empty_data = create_empty_dataset()
    with pytest.raises(ValueError, match="cannot filter empty dataset"):
        filter_by_confidence(empty_data, threshold=0.5)

def test_filter_all_nan_confidence():
    """Test when all confidence values are NaN."""
    data = create_dataset_with_nan_confidence()
    filtered = filter_by_confidence(data, threshold=0.5)
    assert filtered.isnull().all()

def test_interpolate_single_frame():
    """Test interpolation with only 1 frame."""
    data = create_single_frame_dataset()
    result = interpolate_over_time(data)
    assert result.equals(data)  # Should return unchanged

def test_savgol_filter_insufficient_window():
    """Test Savitzky-Golay with window larger than data."""
    data = create_small_dataset(n_frames=5)
    with pytest.raises(ValueError):
        savgol_filter(data, window_length=11)
```

**Complexity:** Medium (12-20 hours for comprehensive coverage)

---

### Gap 4.2: No Stress Tests for Large Datasets
**Location:** New file: `tests/test_stress/`  
**Current State:** Tests use small synthetic data  
**Missing:** Performance tests with realistic large datasets

**Proposed Tests:**
```python
# tests/test_stress/test_large_datasets.py

import pytest

@pytest.mark.stress
def test_load_large_dlc_file():
    """Test loading 1M frame DLC file."""
    file_path = create_large_dlc_file(n_frames=1_000_000)
    start = time.time()
    ds = load_poses.from_dlc_file(file_path)
    duration = time.time() - start
    assert duration < 60  # Should load in <1 minute

@pytest.mark.stress
def test_compute_velocity_large_dataset():
    """Test velocity computation on 100k frames."""
    data = create_large_dataset(n_frames=100_000)
    start = time.time()
    velocity = compute_velocity(data)
    duration = time.time() - start
    assert duration < 10  # Should compute in <10 seconds
```

**Complexity:** Medium (10-15 hours)

---

## 5. Documentation Gaps

### Gap 5.1: Missing Advanced Usage Examples
**Location:** `docs/source/examples/`  
**Current State:** Basic tutorials exist  
**Missing:** Advanced workflow examples

**Needed Examples:**
1. **Multi-individual social interaction analysis**
   - Load data with 2+ individuals
   - Compute inter-individual distances
   - Detect interaction events
   - Visualize social network

2. **Custom coordinate transformations**
   - Define egocentric coordinates
   - Rotate based on body orientation
   - Transform between camera and arena coordinates

3. **Integration with neural data**
   - Load NWB file with pose + spikes
   - Align timestamps
   - Compute behavior-triggered spike averages
   - Plot results

4. **Creating custom napari plugins**
   - Register new layer type
   - Add custom widget
   - Handle user interactions

**Complexity:** Medium (15-25 hours)

---

### Gap 5.2: API Documentation Inconsistencies
**Location:** Various module docstrings  
**Current State:** Most functions documented  
**Missing:** Consistent format, examples in all docstrings

**Standards to Enforce:**
- All public functions have docstrings
- Follow NumPy docstring format
- Include "See Also" section
- Provide at least one example
- List exceptions in "Raises" section

**Example Template:**
```python
def function_name(arg1: Type1, arg2: Type2) -> ReturnType:
    """
    One-line summary.
    
    Extended description of what the function does.
    Can span multiple lines.
    
    Parameters
    ----------
    arg1 : Type1
        Description of arg1.
    arg2 : Type2
        Description of arg2.
        
    Returns
    -------
    ReturnType
        Description of return value.
        
    Raises
    ------
    ValueError
        If input is invalid.
    TypeError
        If argument has wrong type.
        
    See Also
    --------
    related_function : Brief description.
    
    Examples
    --------
    >>> import movement as mv
    >>> data = mv.load_sample_data()
    >>> result = function_name(data, arg2=value)
    >>> print(result)
    [expected output]
    
    Notes
    -----
    Additional information about algorithm, complexity, etc.
    
    References
    ----------
    .. [1] Author (Year). "Title." Journal. DOI.
    """
    pass
```

**Complexity:** Medium (10-20 hours to audit and fix)

---

## 6. Summary Table

| Gap ID | Description | Priority | Complexity | Impact | Estimated Hours |
|--------|-------------|----------|------------|--------|-----------------|
| 1.1 | Outlier detection | High | Medium | High | 8-16 |
| 1.2 | HMM behavior states | High | High | High | 24-40 |
| 1.3 | Interactive 3D viz | Medium-High | Medium | Medium-High | 12-20 |
| 2.1 | Kalman filtering | Medium | Medium | Medium | 10-16 |
| 2.2 | Batch processing | Medium | Low-Medium | Medium | 6-10 |
| 2.3 | Circle/Ellipse ROI | Medium | Low | Medium | 4-8 |
| 2.4 | Error recovery | Medium | Low-Medium | Medium | 8-12 |
| 3.1 | Auto reports | Low-Medium | High | Low-Medium | 20-30 |
| 3.2 | Gait analysis | Low | High | Low | 30+ |
| 3.3 | Type hints | Low | Low | Low | 4-8 |
| 4.1 | Edge case tests | Medium | Medium | Medium | 12-20 |
| 4.2 | Stress tests | Medium | Medium | Medium | 10-15 |
| 5.1 | Advanced examples | Medium | Medium | Medium | 15-25 |
| 5.2 | API docs audit | Low | Medium | Low | 10-20 |

**Total Estimated Effort:** 173-240 hours (~4-6 months part-time)

---

## 7. Recommended Roadmap

### Phase 1: Quick Wins (1-2 months)
- Gap 2.3: Circular/Elliptical ROIs ✅
- Gap 3.3: Type hints audit ✅
- Gap 2.4: Improve error messages ✅
- Gap 4.1: Add edge case tests ✅

### Phase 2: Core Features (2-3 months)
- Gap 1.1: Outlier detection ✅
- Gap 2.1: Kalman filtering ✅
- Gap 2.2: Batch processing ✅
- Gap 5.1: Advanced examples ✅

### Phase 3: Advanced Analytics (3-4 months)
- Gap 1.2: HMM integration ✅
- Gap 1.3: Interactive 3D visualization ✅
- Gap 4.2: Performance benchmarks ✅

### Phase 4: Polish (1-2 months)
- Gap 3.1: Automatic reports ✅
- Gap 5.2: Documentation audit ✅
- Gap 3.2: Gait analysis (optional) ⏸️

---

## 8. How to Use This Document

### For Maintainers
1. Convert gaps to GitHub issues
2. Add labels (priority, complexity, type)
3. Link to project roadmap
4. Assign to milestones

### For Contributors
1. Choose gap matching your skill level
2. Check "Complexity" estimate
3. Review "Files to Modify/Create"
4. Follow code snippets as starting point

### For Researchers
1. Identify gaps affecting your work
2. Comment on relevant issues
3. Provide example use cases
4. Test proposed solutions

---

**Generated:** February 4, 2026  
**Next Review:** May 2026  
**Maintainer:** Core movement team
