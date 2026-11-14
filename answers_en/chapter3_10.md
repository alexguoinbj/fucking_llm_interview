# 3.10 Others — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 1 questions

**1. What Methods Are Used for Multi-Modal Handling of Multi-Resolution Inputs?**

<details>
<summary>💡 Show Answer</summary>

Currently, when processing multi-resolution inputs in multi-modal models, there are two primary strategies:
1. **Tiling Method**: Establish a baseline resolution and split the image into multiple sub-blocks, with each sub-block's shape matching the shape of a tiling unit.
2. **Packing Method**: After splitting, combine image features of different resolutions along the sequence dimension, introducing block-diagonal masks to prevent attention computation between tokens of different images.

</details>

