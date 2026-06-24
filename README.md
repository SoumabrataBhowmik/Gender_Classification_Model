**👤 Gender Classification Under Challenging Conditions – COMSYS-2025 Hackathon** 

<p align="center">
  <img src="Diagram_Gender_Classification_Model.png" width="600" title="Gender_Classification_Model_Architecture">
</p>

Model: ResNet-50 with custom classifier head

Training: Fine-tuned only layer2, layer3, and layer4

Data Augmentation: Applied during training (crop, flip, jitter)

Transform during test: Resize(256) → CenterCrop(224)


📊 Evaluation on Training Set:
  - Recall   : 0.9821
  - F1-Score : 0.9620

📊 Evaluation on Validation Set:
  - Recall   : 0.9679
  - F1-Score : 0.9665


This section details the architecture, preprocessing pipeline, and training configuration for the binary Gender Classification model (Male/Female). 

### System Architecture

```mermaid
flowchart TD
    %% Node Class Definitions for Aesthetics
    classDef inputNode fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#0d47a1,rx:8px,ry:8px
    classDef processNode fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#1b5e20,rx:8px,ry:8px
    classDef modelNode fill:#ffebee,stroke:#d32f2f,stroke-width:2px,color:#b71c1c,rx:8px,ry:8px
    classDef headNode fill:#fff8e1,stroke:#fbc02d,stroke-width:2px,color:#f57f17,rx:8px,ry:8px
    classDef configNode fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#4a148c,rx:8px,ry:8px
    classDef outputNode fill:#e0f7fa,stroke:#0097a7,stroke-width:2px,color:#006064,rx:8px,ry:8px
    
    %% Global Edge styling
    linkStyle default stroke:#78909c,stroke-width:2px

    %% Input Data
    InputData[/"📁 Input Data (Images)<br/><b>Train Data:</b> Male/Female subfolders<br/><b>Val Data:</b> Male/Female subfolders"/]:::inputNode

    %% Preprocessing Subgraph
    subgraph Preprocessing ["🔄 Data Preprocessing (Transformations)"]
        direction LR
        TrainPrep("<b>Train Transformations</b><br/>• RandomResizedCrop (224)<br/>• RandomHorizontalFlip<br/>• RandomRotation (15°)<br/>• ColorJitter<br/>• RandomAffine<br/>• Normalize"):::processNode
        
        ValPrep("<b>Validation Transformations</b><br/>• Resize (256)<br/>• CenterCrop (224)<br/>• Normalize"):::processNode
    end

    %% Architecture Subgraph
    subgraph Architecture ["🧠 Model Architecture (Custom ResNet-50)"]
        direction TB
        Backbone{{"<b>Pretrained ResNet50 Backbone</b><br/>• Frozen Layers: 1st layer<br/>• Unfrozen Layers: 2nd, 3rd, & 4th<br/>• Output: 2048-Dimensional (Flattened to 1D)"}}:::modelNode
        
        Head("<b>Custom Classifier Head</b><br/>Linear (2048 → 1024) + ReLU + Dropout(0.5)<br/>⬇<br/>Linear (1024 → 512) + ReLU + Dropout(0.5)<br/>⬇<br/>Linear (512 → 2) → Output"):::headNode
        
        Backbone ==> Head
    end

    %% Training Configuration
    TrainSetup("⚙️ <b>Training (with CrossEntropyLoss)</b><br/><b>Loss:</b> CrossEntropyLoss (Label Smoothing = 0.1)<br/><b>Optimizer:</b> Adam (lr=1e-4, weight_decay=1e-5)<br/><b>Scheduler:</b> StepLR (step_size=10, gamma=0.1)"):::configNode

    %% Output
    Weights[("💾 Output (Best Model Weights)<br/>Gender_Classification_model.pt")]:::modelNode
    Metrics("🎯 <b>Evaluation Metrics</b><br/>Accuracy, Precision, Recall, F1-Score"):::outputNode

    %% Flow Mapping
    InputData ==> Preprocessing
    Preprocessing ==> Architecture
    Architecture ==> TrainSetup
    TrainSetup ==> Weights
    Weights -.->|"Evaluate"| Metrics
