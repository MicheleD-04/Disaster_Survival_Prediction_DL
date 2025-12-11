<h1 align="center">SafeRoute-GNN: A Graph Neural Network Approach for Disaster Response Routing</h1>

<p align="center"><em>Authors: Bhavya Jain, Michele Daroowala, Pehar Jhamb, Soumya Pandey</em></p>

---

## **1. Understanding of the Problem**

Disaster response routing is critically important for saving lives during emergencies, yet existing navigation systems fail to account for dynamic road blockages caused by floods, earthquakes, landslides, and other disasters.

---

### **Key Challenges**

- **Dynamic Road Blockages:**  
  Traditional routing systems use static road networks and cannot predict which roads will be blocked during disasters. Manual updates from authorities arrive too late or are incomplete.

- **Disaster-Specific Patterns:**  
  Different disasters (floods vs earthquakes vs landslides) cause fundamentally different blockage patterns. A flood blocks low-lying areas near water, while earthquakes damage bridges and overpasses. Simple distance-based models cannot capture these nuances.

- **Spatial Dependencies:**  
  Road blockage risk depends on complex spatial relationships—proximity to rivers, network topology (degree centrality), terrain elevation, and connectivity patterns that traditional methods cannot model.

- **Limited Labeled Data:**  
  Real disaster data with ground-truth blockages is extremely scarce. Emergency agencies rarely maintain comprehensive historical data.

- **Real-Time Requirements:**  
  Emergency routing requires fast inference (seconds) and quick adaptation without manual rule updates.

---

## **2. Understanding of Solutions Proposed by Others**

---

### **2.1 Traditional Shortest Path Algorithms (Dijkstra, A\*)**

#### **Strengths**
- Fast and efficient for static networks  
- Well-established algorithms  
- Provides optimal path computation  
- Works well under normal traffic  

#### **Limitations**
- No awareness of disaster-induced blockages  
- Cannot predict which roads will become unsafe  
- Requires manual updates  
- Treats all roads uniformly  
- No reasoning about spatial hazard patterns  

#### **Gap**
Traditional routing methods lack mechanisms to incorporate **disaster-based spatial risk**.

---

### **2.2 Rule-Based Risk Assessment Systems**

#### **Strengths**
- Interpretable  
- Uses domain expert knowledge  
- No training data needed  
- Simple and fast  

#### **Limitations**
- Cannot model complex spatial dependencies  
- Not adaptable to new disasters  
- Hard to scale  
- Thresholds are arbitrary  
- Miss subtle hazard patterns  

#### **Gap**
Rule systems fail to learn from **real spatial relationships** within road networks.

---

## **3. Dataset Information and Preparation**

Real-world road network data was obtained from **OpenStreetMap (OSM)** using the **OSMnx** library.

---

### **Data Sources**

- **Road Network:**  
  Road lengths, road types, lanes, and geometry from OSM.

- **Water Bodies:**  
  Rivers, streams, lakes—used for flood risk modeling.

- **Shelter Locations:**  
  Hospitals, schools, government buildings.

- **Disaster Epicenter:**  
  User-provided or geocoded coordinates.

---

### **Preprocessing Steps**

1. Download road network.  
2. Convert to undirected graph.  
3. Extract **node features (6D)**.  
4. Extract **edge features (7D)**.  
5. Generate ground-truth blockage labels (simulation).  
6. Make edges bidirectional.  
7. Apply StandardScaler normalization.  

---

### **Dataset Characteristics (Example: Rudraprayag, India)**

- **Nodes:** 300  
- **Edges:** 666 (1332 bidirectional)  
- **Water Features:** 352  
- **Shelters:** 10  

---

### **Node Features (6D)**

- Latitude  
- Longitude  
- Degree centrality  
- Distance to nearest water body  
- Distance to disaster epicenter  
- Elevation  

---

### **Edge Features (7D)**

- Road length  
- Road type encoding  
- Number of lanes  
- Distance to water  
- Disaster intensity score  
- Disaster type encoding  
- Time since disaster started  

---

### **Label Generation Strategy**

- **Flood:** Water proximity thresholds  
- **Earthquake:** Severity-based random damage  
- **Landslide:** Terrain-based blockage probability  
- **Cyclone:** Wide-area impact  
- **Fire:** Clustered near epicenter  
- **Custom:** Universal distance + intensity pattern  

---

## **4. Proposed Method: SafeRoute-GNN**

---

### **4.1 Graph Representation**

Road network modeled as:


- Node features: 6D  
- Edge features: 7D  

---

### **4.2 Graph Attention Network (GAT) Architecture**

| Layer | Dimensions | Heads | Dropout |
|-------|------------|--------|----------|
| Layer 1 | 6 → 256 | 4 | 0.3 |
| Layer 2 | 256 → 256 | 4 | 0.3 |
| Layer 3 | 256 → 128 | 2 | 0.3 |

#### **Why GAT?**
- Learns importance weights for neighbors  
- Captures varying spatial relationships  
- More expressive than GCN  

---

### **4.3 Edge Blockage Prediction**

1. Concatenate node embeddings of endpoints.  
2. Pass through MLP (256 → 128 → 64 → 1).  
3. Sigmoid output: Probability of blockage.  

---

### **4.4 Route Planning Process**

1. Input user location + disaster details  
2. Predict edge blockage probabilities  
3. Remove edges above threshold  
4. Build safe graph  
5. Run Dijkstra to shelter  
6. Return shortest safe path  
7. Visualize results  

---

## **5. Experimentation and Ablation Studies**

---

### **5.1 Baseline: Standard GCN**

- **Accuracy:** 82%  
- **F1:** 75.3%  
- **Loss:** 0.45  

GCN underperforms due to uniform weighting of neighbors.

---

### **5.2 Ablation: Remove Disaster Features**

- **Accuracy:** 79%  
- **F1:** 71.8%  
- **Loss:** 0.48  

Disaster features contribute significantly to performance (+9%).

---

### **5.3 Ablation: Vary GAT Layers**

| Model | Accuracy | F1 | Loss | Training Time |
|--------|-----------|--------|--------|----------------|
| 2 Layers | 83% | 77.1% | 0.43 | 1.5 min |
| **3 Layers (Final)** | **88%** | **83.3%** | **0.41** | **2.2 min** |
| 4 Layers | 87% | 82.5% | 0.42 | 3.8 min |
| 5 Layers | 86% | 81.9% | 0.43 | 5.1 min |

---

## **6. Results**

---

### **Model Comparison**

| Model | Accuracy | F1 | Loss | Blocked Predicted |
|--------|-----------|--------|--------|------------------|
| GCN Baseline | 82% | 75.3% | 0.45 | 95/666 |
| SafeRoute-GNN (No Disaster) | 79% | 71.8% | 0.48 | 78/666 |
| SafeRoute-GNN (2 Layers) | 83% | 77.1% | 0.43 | 89/666 |
| **SafeRoute-GNN (Final)** | **88%** | **83.3%** | **0.41** | **102/666** |

---

### **Example Evaluation Scenario**

- **Disaster:** Earthquake (Medium Severity)  
- **Intensity:** 8.8  
- **Blocked Roads:** 102 (15.3%)  

#### **Routing Result**
- **Start:** Kedarnath Temple  
- **Shelter:** Jankalyan Hospital  
- **Distance:** 42.75 km  
- **Estimated Time:** 85 minutes  
- **Segments:** 15  
- **Safety:** Avoids all high-risk segments  

---

## **7. Lessons Learnt**

- Graph structure enables deeper spatial reasoning  
- Attention improves interpretability  
- Disaster features are crucial  
- Simulated labels work for prototyping  
- Deeper GNN ≠ always better  
- Simple models scale better in real-world systems  
- Ablation studies reveal importance of components  
- OSM enables global scalability  

---

## **8. Future Work and Limitations**

---

### **Limitations**

- Simulated labels instead of real data  
- Static disaster assumptions  
- No traffic modeling  
- Limited geographic testing  
- No uncertainty quantification  

---

### **Future Directions**

- Real disaster datasets from agencies  
- Temporal GNNs  
- Multi-agent evacuation simulation  
- Satellite imagery integration  
- Mobile app deployment  
- Multi-city benchmarking  
- Explainability dashboards  

---

## **9. References**

- Veličković et al., Graph Attention Networks  
- Kipf & Welling, Graph Convolutional Networks  
- Hamilton et al., GraphSAGE  
- Litman, Lessons from Katrina & Rita  
- Cova & Johnson, Lane-Based Evacuation Routing  
- Boeing, OSMnx  
- OpenStreetMap Contributors  
- PyTorch Geometric  
- PyTorch Deep Learning Library  

---

## **10. Conclusion**

SafeRoute-GNN effectively predicts disaster-induced road blockages using:

- Graph Attention Networks  
- Disaster-aware features  
- Real-world OSM data  
- Fast real-time inference  

**Impact:** Enables safer evacuation by avoiding dangerous roads and guiding users to shelters efficiently.

---
