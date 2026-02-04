# Movement Project: Task Breakdown by Skill Level

**Purpose:** This document provides **immediately actionable tasks** for contributors at different skill levels. Each task is self-contained with clear success criteria.

---

## 🟢 Level 1: Beginner Tasks (1-4 hours each)

Perfect for first-time contributors! These tasks require basic Python knowledge and help you learn the codebase structure.

---

### Task 1.1: Add Examples to Vector Utility Functions

**Goal:** Add usage examples to docstrings in `movement/utils/vector.py`

**Skills:** Python basics, NumPy arrays, docstring formatting

**Files:**
- `movement/utils/vector.py`

**Steps:**
1. Open `movement/utils/vector.py`
2. Find functions without "Examples" section in docstrings
3. Add examples following NumPy docstring format
4. Test examples with `python -m doctest movement/utils/vector.py`

**Example:**
```python
def cart2pol(x: np.ndarray, y: np.ndarray) -> tuple[np.ndarray, np.ndarray]:
    """
    Convert Cartesian coordinates to polar coordinates.
    
    ... (existing docstring) ...
    
    Examples
    --------
    >>> import numpy as np
    >>> x = np.array([1.0, 0.0, -1.0])
    >>> y = np.array([0.0, 1.0, 0.0])
    >>> rho, phi = cart2pol(x, y)
    >>> print(rho)
    [1. 1. 1.]
    >>> print(phi)  # in radians
    [0.         1.57079633 3.14159265]
    """
```

**Success Criteria:**
- [ ] All public functions have at least one example
- [ ] Examples are runnable and produce expected output
- [ ] Doctest passes without errors

**Related Issue:** Good first issue for documentation

---

### Task 1.2: Improve CLI Help Messages

**Goal:** Enhance `--help` output for the movement CLI tool

**Skills:** Python argparse, command-line interfaces

**Files:**
- `movement/cli_entrypoint.py`

**Current State:**
```bash
$ movement --help
# Minimal help text
```

**Desired State:**
```bash
$ movement --help
usage: movement [-h] {load,filter,compute,plot} ...

Movement: Analyze animal body movements across space and time

positional arguments:
  {load,filter,compute,plot}
    load                Load pose data from various formats
    filter              Clean and smooth tracking data
    compute             Calculate kinematic variables
    plot                Visualize trajectories and occupancy

optional arguments:
  -h, --help            show this help message and exit

Examples:
  movement load --format dlc data.h5
  movement filter --method savgol data.nc --output filtered.nc
  movement compute velocity data.nc --output velocity.nc

For more information: https://movement.neuroinformatics.dev
```

**Steps:**
1. Add detailed help text to argument parser
2. Add usage examples to main help
3. Add help for each subcommand
4. Test with `movement --help` and `movement load --help`

**Success Criteria:**
- [ ] Help text includes description, examples, and URL
- [ ] Each subcommand has detailed help
- [ ] Examples are copy-pasteable

---

### Task 1.3: Fix Spelling Errors in Codebase

**Goal:** Run codespell and fix typos in comments/docstrings

**Skills:** Basic text editing

**Files:** All `.py` files in `movement/`

**Steps:**
```bash
# 1. Run codespell
codespell movement/ tests/ --skip="*.git,*.pyc"

# 2. Fix reported typos manually
# 3. Add false positives to .codespellrc

# 4. Verify
codespell movement/ tests/
```

**Common Typos to Watch For:**
- "seperator" → "separator"
- "occured" → "occurred"
- "recieve" → "receive"
- "paramter" → "parameter"

**Success Criteria:**
- [ ] `codespell` runs with 0 errors
- [ ] All real typos fixed
- [ ] False positives added to `.codespellrc`

---

### Task 1.4: Add Type Hints to `transforms.py`

**Goal:** Add type annotations to functions in `movement/transforms.py`

**Skills:** Python type hints, numpy arrays

**Files:**
- `movement/transforms.py`

**Before:**
```python
def scale(data, factor):
    """Scale spatial coordinates by a factor."""
    return data * factor
```

**After:**
```python
import xarray as xr

def scale(data: xr.DataArray, factor: float) -> xr.DataArray:
    """Scale spatial coordinates by a factor."""
    return data * factor
```

**Steps:**
1. Import necessary types from `typing`, `numpy`, `xarray`
2. Add type hints to all function parameters and returns
3. Run `mypy movement/transforms.py` to verify
4. Fix any type errors

**Success Criteria:**
- [ ] All functions have type hints
- [ ] `mypy` passes with no errors
- [ ] Type hints match actual usage

---

### Task 1.5: Create "Quick Start" Tutorial

**Goal:** Write a 5-minute tutorial for new users

**Skills:** Python, markdown, technical writing

**Files:**
- `docs/source/examples/quickstart.md` (new file)

**Content Structure:**
```markdown
# Quick Start: Analyze Your First Tracking Dataset

Learn how to load, visualize, and analyze pose tracking data in 5 minutes!

## 1. Installation
\`\`\`bash
conda install -c conda-forge movement
\`\`\`

## 2. Load Sample Data
\`\`\`python
import movement as mv

# Load built-in example
ds = mv.load_sample_data("DLC_single-mouse")
print(ds)
\`\`\`

## 3. Visualize Trajectory
\`\`\`python
from movement.plots import plot_centroid_trajectory

# Plot nose keypoint trajectory
plot_centroid_trajectory(ds, keypoint="nose")
\`\`\`

## 4. Compute Velocity
\`\`\`python
from movement.kinematics import compute_velocity

# Calculate velocity in pixels/second
velocity = compute_velocity(ds, fps=30)
print(f"Mean speed: {velocity.mean():.2f} px/s")
\`\`\`

## Next Steps
- [Load your own data](../user_guide/input_output.html)
- [Apply filters](../api/filtering.html)
- [Explore GUI](../user_guide/gui.html)
```

**Success Criteria:**
- [ ] Tutorial runs without errors on fresh install
- [ ] Takes <5 minutes to complete
- [ ] Linked from main documentation index

---

### Task 1.6: Add Edge Case Test for Empty Dataset

**Goal:** Test filtering functions with empty datasets

**Skills:** Python, pytest

**Files:**
- `tests/test_unit/test_filtering.py`

**Example Test:**
```python
import pytest
import xarray as xr
import numpy as np
from movement.filtering import filter_by_confidence

def test_filter_by_confidence_empty_dataset():
    """Test that filtering empty dataset raises appropriate error."""
    # Create empty dataset (0 time points)
    empty_ds = xr.Dataset(
        {
            "position": xr.DataArray(
                np.empty((0, 2, 1, 2)),  # (time=0, space, individuals, keypoints)
                dims=["time", "space", "individuals", "keypoints"],
            ),
            "confidence": xr.DataArray(
                np.empty((0, 1, 2)),
                dims=["time", "individuals", "keypoints"],
            ),
        }
    )
    
    # Should raise ValueError with helpful message
    with pytest.raises(ValueError, match="Cannot filter empty dataset"):
        filter_by_confidence(empty_ds, threshold=0.5)
```

**Tasks:**
- Add tests for: empty datasets, single-frame datasets, all-NaN data
- Test for: `filter_by_confidence`, `interpolate_over_time`, `savgol_filter`

**Success Criteria:**
- [ ] 3+ edge case tests added per filter function
- [ ] Tests fail initially (demonstrating gap)
- [ ] Functions modified to handle edge cases
- [ ] All tests pass

---

## 🟡 Level 2: Intermediate Tasks (4-16 hours each)

For contributors with Python experience and familiarity with scientific computing libraries.

---

### Task 2.1: Implement Circular ROI Class

**Goal:** Add support for circular regions of interest

**Skills:** Python OOP, geometry, shapely

**Files:**
- `movement/roi/circle.py` (new)
- `tests/test_unit/test_roi/test_circle.py` (new)
- `examples/circular_roi_example.py` (new)

**Implementation:**
```python
# movement/roi/circle.py

from __future__ import annotations
import numpy as np
import xarray as xr
from shapely.geometry import Point
from attrs import define, field

@define
class CircularROI:
    """Circular region of interest.
    
    Parameters
    ----------
    center : tuple of float
        (x, y) coordinates of circle center in spatial units
    radius : float
        Circle radius in spatial units
        
    Examples
    --------
    >>> roi = CircularROI(center=(0, 0), radius=10)
    >>> points = np.array([[0, 0], [5, 5], [15, 0]])
    >>> roi.contains_points(points)
    array([ True,  True, False])
    """
    
    center: tuple[float, float] = field()
    radius: float = field()
    
    @radius.validator
    def _validate_radius(self, attribute, value):
        if value <= 0:
            raise ValueError("Radius must be positive")
    
    @property
    def area(self) -> float:
        """Calculate circle area."""
        return np.pi * self.radius ** 2
    
    def contains_points(self, points: xr.DataArray | np.ndarray) -> xr.DataArray | np.ndarray:
        """
        Check if points are inside the circle.
        
        Parameters
        ----------
        points : xarray.DataArray or numpy.ndarray
            Points with shape (..., 2) where last dimension is [x, y]
            
        Returns
        -------
        mask : same type as input
            Boolean mask where True means point is inside circle
        """
        points_arr = points.values if isinstance(points, xr.DataArray) else points
        
        # Compute distances from center
        dx = points_arr[..., 0] - self.center[0]
        dy = points_arr[..., 1] - self.center[1]
        distances = np.sqrt(dx**2 + dy**2)
        
        mask = distances <= self.radius
        
        if isinstance(points, xr.DataArray):
            return xr.DataArray(mask, coords=points.coords, dims=points.dims[:-1])
        return mask
    
    def compute_occupancy(self, trajectory: xr.DataArray) -> float:
        """
        Calculate fraction of time spent inside circle.
        
        Parameters
        ----------
        trajectory : xarray.DataArray
            Trajectory data with shape (time, space)
            
        Returns
        -------
        occupancy : float
            Fraction between 0 and 1
        """
        mask = self.contains_points(trajectory)
        return float(mask.sum() / len(mask))
    
    def to_polygon(self, n_points: int = 32):
        """Convert to polygon approximation (for plotting)."""
        from shapely.geometry import Point
        return Point(self.center).buffer(self.radius, resolution=n_points)
    
    def plot(self, ax=None, **kwargs):
        """Plot the circular ROI."""
        import matplotlib.pyplot as plt
        from matplotlib.patches import Circle
        
        if ax is None:
            ax = plt.gca()
        
        circle = Circle(self.center, self.radius, fill=False, **kwargs)
        ax.add_patch(circle)
        return ax
```

**Test Plan:**
```python
# tests/test_unit/test_roi/test_circle.py

import pytest
import numpy as np
from movement.roi.circle import CircularROI

def test_circular_roi_creation():
    roi = CircularROI(center=(0, 0), radius=10)
    assert roi.center == (0, 0)
    assert roi.radius == 10
    assert roi.area == pytest.approx(314.159, rel=1e-3)

def test_circular_roi_invalid_radius():
    with pytest.raises(ValueError, match="Radius must be positive"):
        CircularROI(center=(0, 0), radius=-5)

def test_contains_points_inside():
    roi = CircularROI(center=(0, 0), radius=10)
    points = np.array([[0, 0], [5, 5], [7.07, 7.07]])  # all inside
    mask = roi.contains_points(points)
    assert np.all(mask)

def test_contains_points_outside():
    roi = CircularROI(center=(0, 0), radius=10)
    points = np.array([[20, 0], [0, 20], [15, 15]])  # all outside
    mask = roi.contains_points(points)
    assert not np.any(mask)

def test_contains_points_boundary():
    roi = CircularROI(center=(0, 0), radius=10)
    points = np.array([[10, 0], [0, 10], [-10, 0]])  # on boundary
    mask = roi.contains_points(points)
    assert np.all(mask)  # boundary counts as inside

def test_compute_occupancy():
    roi = CircularROI(center=(0, 0), radius=10)
    # Create trajectory: half inside, half outside
    trajectory = xr.DataArray(
        np.array([[0, 0], [5, 0], [15, 0], [20, 0]]),
        dims=["time", "space"],
    )
    occupancy = roi.compute_occupancy(trajectory)
    assert occupancy == 0.5  # 2 out of 4 points inside
```

**Success Criteria:**
- [ ] Class implemented with all methods
- [ ] Full test coverage (>90%)
- [ ] Example script demonstrates usage
- [ ] Documentation added to API reference
- [ ] Works with xarray and numpy inputs

---

### Task 2.2: Add Batch Processing Function

**Goal:** Create utility for processing multiple files in parallel

**Skills:** Python multiprocessing, functional programming

**Files:**
- `movement/utils/batch.py` (new)
- `tests/test_unit/test_batch.py` (new)

**Implementation:**
```python
# movement/utils/batch.py

from __future__ import annotations
from typing import Callable, List, Any
from pathlib import Path
import multiprocessing as mp
from functools import partial
from tqdm import tqdm
import logging

logger = logging.getLogger(__name__)

def process_files_parallel(
    file_paths: List[Path | str],
    processing_func: Callable[[Path], Any],
    n_jobs: int = -1,
    show_progress: bool = True,
    return_exceptions: bool = False,
    **func_kwargs,
) -> List[Any]:
    """
    Process multiple files in parallel.
    
    Parameters
    ----------
    file_paths : list of Path or str
        Input files to process
    processing_func : callable
        Function to apply to each file. Should accept Path as first argument.
    n_jobs : int, default=-1
        Number of parallel jobs. -1 uses all available CPUs.
    show_progress : bool, default=True
        Whether to show progress bar
    return_exceptions : bool, default=False
        If True, exceptions are returned instead of raised
    **func_kwargs
        Additional keyword arguments passed to processing_func
        
    Returns
    -------
    results : list
        Results from processing each file, in same order as input
        
    Examples
    --------
    >>> from movement import load_poses
    >>> 
    >>> def load_and_filter(file_path, threshold=0.6):
    ...     ds = load_poses.from_dlc_file(file_path)
    ...     return filter_by_confidence(ds, threshold=threshold)
    >>> 
    >>> files = ["file1.h5", "file2.h5", "file3.h5"]
    >>> results = process_files_parallel(files, load_and_filter, threshold=0.7)
    """
    # Convert to Path objects
    file_paths = [Path(f) for f in file_paths]
    
    # Determine number of jobs
    if n_jobs == -1:
        n_jobs = mp.cpu_count()
    elif n_jobs < 1:
        raise ValueError("n_jobs must be -1 or positive integer")
    
    # If only 1 job or 1 file, process sequentially
    if n_jobs == 1 or len(file_paths) == 1:
        return _process_sequential(
            file_paths, processing_func, show_progress, return_exceptions, **func_kwargs
        )
    
    # Parallel processing
    with mp.Pool(n_jobs) as pool:
        func_partial = partial(processing_func, **func_kwargs)
        
        if show_progress:
            results = list(tqdm(
                pool.imap(func_partial, file_paths),
                total=len(file_paths),
                desc="Processing files",
            ))
        else:
            results = pool.map(func_partial, file_paths)
    
    return results


def _process_sequential(file_paths, func, show_progress, return_exceptions, **kwargs):
    """Helper for sequential processing."""
    results = []
    iterator = tqdm(file_paths, desc="Processing files") if show_progress else file_paths
    
    for file_path in iterator:
        try:
            result = func(file_path, **kwargs)
            results.append(result)
        except Exception as e:
            if return_exceptions:
                results.append(e)
            else:
                logger.error(f"Failed to process {file_path}: {e}")
                raise
    
    return results


def process_files_sequential(
    file_paths: List[Path | str],
    processing_func: Callable[[Path], Any],
    show_progress: bool = True,
    **func_kwargs,
) -> List[Any]:
    """
    Process multiple files sequentially (no parallelization).
    
    Useful for debugging or when function is not picklable.
    
    Parameters
    ----------
    Same as process_files_parallel, except no n_jobs parameter.
    """
    file_paths = [Path(f) for f in file_paths]
    return _process_sequential(file_paths, processing_func, show_progress, False, **func_kwargs)
```

**Success Criteria:**
- [ ] Parallel processing works correctly
- [ ] Progress bar displays accurately
- [ ] Error handling works (with/without return_exceptions)
- [ ] Tests verify results match sequential processing
- [ ] Works with various processing functions

---

### Task 2.3: Implement Outlier Detection

**Goal:** Add statistical outlier detection to filtering module

**Skills:** Statistics, NumPy, SciPy

**Files:**
- `movement/filtering.py`
- `tests/test_unit/test_filtering.py`

**Implementation Outline:**
```python
# Add to movement/filtering.py

from typing import Literal
import numpy as np
from scipy import stats
from sklearn.ensemble import IsolationForest  # optional

def detect_outliers(
    data: xr.DataArray,
    method: Literal["iqr", "zscore", "isolation_forest"] = "iqr",
    threshold: float | None = None,
    window: int | None = None,
) -> xr.DataArray:
    """
    Detect outliers in pose tracking data.
    
    Three methods available:
    1. IQR: Interquartile range method (robust to outliers)
    2. Z-score: Standard deviations from mean
    3. Isolation Forest: ML-based anomaly detection
    
    Parameters
    ----------
    data : xarray.DataArray
        Input tracking data with 'time' dimension
    method : {'iqr', 'zscore', 'isolation_forest'}
        Detection method
    threshold : float, optional
        Method-specific threshold:
        - iqr: multiplier for IQR (default=1.5)
        - zscore: number of std devs (default=3.0)
        - isolation_forest: contamination fraction (default=0.1)
    window : int, optional
        If provided, detect outliers within rolling windows
        
    Returns
    -------
    outliers : xarray.DataArray (bool)
        Boolean mask where True indicates outliers
        
    Examples
    --------
    >>> data = load_sample_data()
    >>> outliers = detect_outliers(data.position, method="iqr")
    >>> print(f"Found {outliers.sum()} outliers")
    >>> 
    >>> # Replace outliers with NaN
    >>> clean_data = data.where(~outliers)
    """
    if method == "iqr":
        return _detect_outliers_iqr(data, threshold or 1.5, window)
    elif method == "zscore":
        return _detect_outliers_zscore(data, threshold or 3.0, window)
    elif method == "isolation_forest":
        return _detect_outliers_iforest(data, threshold or 0.1)
    else:
        raise ValueError(f"Unknown method: {method}")
```

**(Include helper functions `_detect_outliers_iqr`, etc.)**

**Success Criteria:**
- [ ] Three methods implemented and tested
- [ ] Works with multidimensional data (time, space, individuals, keypoints)
- [ ] Rolling window option works correctly
- [ ] Comprehensive tests with known outliers
- [ ] Example notebook demonstrating usage

---

## 🟠 Level 3: Advanced Tasks (16-40 hours each)

For experienced contributors comfortable with complex algorithms and system design.

---

### Task 3.1: Implement Hidden Markov Model (HMM) Module

**Goal:** Create behavior state detection using HMMs

**Skills:** Machine learning, statistical modeling, scikit-learn/hmmlearn

**Files:**
- `movement/analysis/` (new directory)
- `movement/analysis/__init__.py`
- `movement/analysis/hmm.py`
- `tests/test_unit/test_analysis/test_hmm.py`
- `examples/hmm_behavior_classification.py`

**Key Components:**
1. `BehaviorHMM` class wrapping hmmlearn
2. Feature extraction from kinematics
3. Model fitting and prediction
4. Visualization of state sequences
5. Model evaluation metrics

**Success Criteria:**
- [ ] Can train HMM on kinematic features
- [ ] Predicts behavior states for new data
- [ ] Export/import trained models
- [ ] Comprehensive example with real data
- [ ] Documentation with theory background

---

### Task 3.2: Add Interactive 3D Visualization

**Goal:** Create rich 3D trajectory viewer with Plotly

**Skills:** 3D graphics, Plotly, animation

**Files:**
- `movement/plots/trajectory_3d.py` (new)
- `tests/test_unit/test_plots/test_trajectory_3d.py`
- `examples/3d_visualization_example.py`

**Features:**
- Interactive rotation, zoom, pan
- Color by time, speed, or state
- Animation along time axis
- Multiple individuals overlay
- Export to HTML/GIF

**Success Criteria:**
- [ ] Works with 3D pose data
- [ ] Smooth animations (>30 fps)
- [ ] Export to standalone HTML
- [ ] Customizable appearance
- [ ] Example with multi-animal data

---

## 🔴 Level 4: Expert Tasks (40+ hours each)

For expert contributors ready to tackle architectural challenges.

---

### Task 4.1: Add GPU Acceleration

**Goal:** Accelerate kinematic computations using CuPy

**Skills:** CUDA, CuPy, performance optimization

**Files:**
- `movement/kinematics/*.py` (modify all)
- `movement/utils/gpu.py` (new)

**Approach:**
- Auto-detect CUDA availability
- Seamless fallback to NumPy
- GPU-accelerated velocity, acceleration, distances
- Memory management for large datasets

**Success Criteria:**
- [ ] 10-100x speedup on large datasets
- [ ] No API changes (transparent acceleration)
- [ ] Comprehensive benchmarks
- [ ] CI tests on GPU runners

---

### Task 4.2: Implement Plugin System

**Goal:** Allow users to register custom analysis functions

**Skills:** Python entry points, architecture design

**Files:**
- `movement/plugins/` (new)
- `movement/plugins/registry.py`
- Documentation for plugin development

**Features:**
- Entry point discovery (like napari)
- Plugin validation
- Auto-generation of API docs
- Example plugin template

**Success Criteria:**
- [ ] External packages can register plugins
- [ ] Plugins discoverable via CLI/API
- [ ] Comprehensive developer guide
- [ ] Example plugin repository

---

## Summary Table

| Level | Tasks | Total Hours | Skills Required |
|-------|-------|-------------|-----------------|
| 1 (Beginner) | 6 tasks | 6-24 hours | Python basics, git, documentation |
| 2 (Intermediate) | 3 tasks | 12-48 hours | NumPy, xarray, testing, algorithms |
| 3 (Advanced) | 2 tasks | 32-80 hours | ML, optimization, visualization |
| 4 (Expert) | 2 tasks | 80+ hours | CUDA, architecture, system design |

---

## How to Claim a Task

1. **Choose a task** matching your skill level
2. **Check GitHub issues** - task may already be assigned
3. **Comment on issue** or create new one with task ID
4. **Fork repository** and create feature branch
5. **Implement task** following success criteria
6. **Submit pull request** with:
   - Link to task in this document
   - Description of implementation
   - Tests demonstrating functionality

---

## Questions?

- [Zulip Chat](https://neuroinformatics.zulipchat.com/#narrow/stream/406001-Movement)
- [Contributing Guide](CONTRIBUTING.md)
- [Documentation](https://movement.neuroinformatics.dev/)

**Happy contributing!** 🚀
