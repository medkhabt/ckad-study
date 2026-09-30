find pods with team label artemidis or aircontrol and a label of tier backend.
`kubectl get pods -n ckad -l 'team in (artemidis,aircontrol)',tier=backend`
