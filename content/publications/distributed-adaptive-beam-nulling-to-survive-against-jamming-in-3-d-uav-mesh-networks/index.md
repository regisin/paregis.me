---
title: "Distributed Adaptive Beam Nulling to Survive Against Jamming in 3 D Uav Mesh Networks"
date: 2018-06-01T00:00:00-05:00
draft: false
citation: "Bhunia, S., **Regis, P. A.**, & Sengupta, S. (2018). Distributed adaptive beam nulling to survive against jamming in 3d uav mesh networks. Computer Networks, 137, 83-97."
categories:
- Journal
---

## Abstract
In the future generation mission-centric tactical network, 3D UAV mesh network will play a crucial role for its several advantages. However, any adversarial node with the 3D movement capability poses a significant threat to these networks as the adversary can position itself to attack the crucial links. An adaptive beamnulling antenna is used to spatially filter out signal coming from a certain direction that can maintain the links active without requiring additional spectrum. However, determining the beamnull region is very challenging for jammer with mobility. This paper presents a distributed mechanism where nodes measure the jammer’s direction in a discrete interval and determine the optimal beamnull for the next interval. Kalman filter based tracking mechanism is used to estimate the most likely trajectory of the jammer from noisy observation of the jammer’s position. A beam null border is determined by calculating confidence region of jammer’s current and next position estimates. An optimization goal is presented to determine the optimal beam null that minimizes the number of deactivated links while maximizing the higher value of confidence for keeping the jammer inside the null. The framework works in the physical layer and can work with any upper layer protocol. The survivability of a 3D mesh network with a mobile jammer is studied through simulation that validates an 96.65% reduction in the number of jammed nodes.

## Resources
- [Manuscript](resources/comnet_18.pdf)

## Bibtex
```latex
@article{bhunia2018distributed,
    title={Distributed Adaptive Beam Nulling to Survive Against Jamming in 3D UAV Mesh Networks},
    author={Suman Bhunia, Paulo Alexandre Regis, and Shamik Sengupta},
    year={2018},
    month={June},
    doi={10.1016/j.comnet.2018.03.011},
    journal={Elsevier Computer Networks},
}
```