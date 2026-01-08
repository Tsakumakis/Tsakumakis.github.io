
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <script src="https://aframe.io/releases/1.7.0/aframe.min.js"></script>
    <style>
      #rubiks-menu {
        position: absolute;
        top: 20px;
        left: 20px;
        z-index: 10;
        background: rgba(255,255,255,0.85);
        padding: 12px;
        border-radius:8px;
        font-family: sans-serif;
      }
      #rubiks-menu button { margin: 2px 4px; padding: 6px 12px; border-radius: 5px; border: none; background: #6a95ff; color: white; font-weight: bold; cursor: pointer;}
      #rubiks-menu button:hover { background: #4562b5;}
    </style>
  </head>
  <body>
    <div id="rubiks-menu">
      <b>Rotate Slice (grouped, centered):</b><br>
      <button onclick="rotateSlice('U')">U (Top, y=3)</button>
      <button onclick="rotateSlice('D')">D (Bottom, y=1)</button>
      <button onclick="rotateSlice('L')">L (Left, x=-1)</button>
      <button onclick="rotateSlice('R')">R (Right, x=1)</button>
      <button onclick="rotateSlice('M')">M (middle, x=0)</button>
      <button onclick="rotateSlice('F')">F (Front, z=-2)</button>
      <button onclick="rotateSlice('B')">B (Back, z=-4)</button>
    </div>
    <a-scene stats>
      <a-assets>
        <a-asset-item id="cube-model" src="https://kostasgian21.github.io/computer_graphics/rubiks_cube_standard_solid_v3.glb"></a-asset-item>
        <a-asset-item id="bench" src="https://Tsakumakis.github.io/Picnic_bench.glb"></a-asset-item>
        <a-asset-item id="name-model" src="https://Tsakumakis.github.io/3dname.glb"></a-asset-item>      
        <img id="skyTexture" src="https://Tsakumakis.github.io/backround_image" crossorigin="anonymous"/>
      </a-assets>
      <a-sky src="#skyTexture"></a-sky>
      <a-entity light="type: directional; intensity:2; color:#fff" position="1 4 3" rotation="0 90 0"></a-entity>
      <a-entity light="type: directional; intensity:5; color:#fff" position="-2 4 -3" rotation="0 -90 0"></a-entity>
      <a-entity light="type: directional; intensity:5; color:#fff" position="0 3 0" rotation="0 0 0"></a-entity>
      <a-entity light="type: directional; intensity:5; color:#fff" position="0 -1 0" rotation="90 0 0"></a-entity>  
      <a-entity light="type: ambient; intensity:2.35; color:#fff"></a-entity>
      <a-entity light="type: point; intensity:5; distance:10" position="-3 2 0"></a-entity>
      <a-entity light="type: point; intensity:5; distance:10" position="0 5 0"></a-entity>
      <a-entity light="type: point; intensity:5; distance:10" position="3 2 0"></a-entity>
      <a-entity light="type: point; intensity:5; distance:10" position="0 1 -5"></a-entity>
      <a-entity shadow="cast: true; receive: true"></a-entity>
      <a-entity id="cube-cluster"></a-entity>
<!-- name-->
<a-entity gltf-model="#name-model"
          position="0 5 -3"
          scale="50 50 50"
          rotation="90 90 90">
</a-entity>
<!-- bench -->
<a-entity gltf-model="#bench"
          position="0.4 -2.5 -2.35"
          scale="125 100 200"
          rotation="0 90 0">
</a-entity>
    <!-- camera controls -->
    <a-entity id="camera" camera position="0 3 5" look-controls wasd-controls="acceleration:100"></a-entity>
  <!-- dude sitting -->
  <a-box position="0 1 -7" rotation="10 0 0" scale="3 7 2" color="#BA1414"></a-box>
  <a-box position="-1.6 1.6 -7" rotation="10 0 0" scale="1 5.5 2" color="#F1C27D"></a-box>
  <a-box position="1.6 1.6 -7" rotation="10 0 0" scale="1 5.5 2" color="#F1C27D"></a-box>
  <a-box position="0 5.1 -6.3" rotation="10 0 0" scale="2 2 2" color="#F1C27D"></a-box>
  <a-box position="0.5 5.2 -5.3" rotation="10 0 0" scale="0.2 0.5 0.3" color="black"></a-box>
  <a-box position="-0.5 5.2 -5.3" rotation="10 0 0" scale="0.2 0.5 0.3" color="black"></a-box>
  <a-box position="0 4.5 -5.3" rotation="10 0 0" scale="0.6 0.1 0.3" color="black"></a-box>
  <a-box position="0 6 -6.15" rotation="10 0 0" scale="2.5 0.8 2.2" color="grey"></a-box>
  <a-box position="0 6.5 -6.15" rotation="10 0 0" scale="2 0.8 1.8" color="grey"></a-box>
  <script>
    // Cube parameters
    const positions = [-1, 0, 1];
    const layersY = [1, 2, 3];
    const layersZ = [-2, -3, -4];
    const cubeSize = 0.9;
    const model = '#cube-model';

    // Slices definitions, with their centers and match functions
    const sliceDefs = {
      U:  {center: [0,3,-3],  match: (x,y,z) => y===3, axis: 'y', angle: 90},      // Top row
      D:  {center: [0,1,-3],  match: (x,y,z) => y===1, axis: 'y', angle: -90},     // Bottom row
      L:  {center: [-1,2,-3], match: (x,y,z) => x===-1, axis: 'x', angle: -90},    // Left column
      R:  {center: [1,2,-3],  match: (x,y,z) => x===1, axis: 'x', angle: 90},      // Right column
      M: {center: [0,2,-3],  match: (x,y,z) => x===0, axis: 'x', angle: -90},     // Middle column
      F:  {center: [0,2,-2],  match: (x,y,z) => z===-2, axis: 'z', angle: 90},     // Front slice
      B:  {center: [0,2,-4],  match: (x,y,z) => z===-4, axis: 'z', angle: -90},    // Back slice
    };

    // Create all cubelets under cube-cluster, no duplicates
    const cluster = document.getElementById('cube-cluster');
    let cubelets = [];
    for (let y of layersY) {
      for (let x of positions) {
        for (let z of layersZ) {
          let cubelet = document.createElement('a-entity');
          cubelet.setAttribute('gltf-model', model);
          cubelet.setAttribute('position', `${x} ${y} ${z}`);
          cubelet.setAttribute('scale', `${cubeSize} ${cubeSize} ${cubeSize}`);
          cubelet.setAttribute('rotation', `0 0 0`);
          cubelet.setAttribute('data-x', x);
          cubelet.setAttribute('data-y', y);
          cubelet.setAttribute('data-z', z);
          cluster.appendChild(cubelet);
          cubelets.push(cubelet);
        }
      }
    }

    // Utility: Convert string rotation to object
    function parseRotation(rot) {
      if (typeof rot === 'string') {
        let arr = rot.trim().split(/\s+/).map(Number);
        return {x: arr[0]||0, y: arr[1]||0, z: arr[2]||0};
      }
      return rot || {x:0, y:0, z:0};
    }

    // Rotate slice animation
    function rotateSlice(face) {
      const sliceDef = sliceDefs[face];
      if (!sliceDef) return;
      const {center, match, axis, angle} = sliceDef;

      // Find affected cubelets
      let affected = cubelets.filter(cubelet => {
        const x = parseInt(cubelet.getAttribute('data-x'));
        const y = parseInt(cubelet.getAttribute('data-y'));
        const z = parseInt(cubelet.getAttribute('data-z'));
        return match(x,y,z);
      });

      // For each affected cubelet, animate rotation about the slice center
      affected.forEach(cubelet => {
        // Get current position and rotation
        let pos = cubelet.getAttribute('position');
        let rot = parseRotation(cubelet.getAttribute('rotation'));

        // Compute position relative to slice center
        let rel = {
          x: pos.x - center[0],
          y: pos.y - center[1],
          z: pos.z - center[2]
        };

        // Rotate rel position around axis by angle
        let rad = angle * Math.PI/180;
        let newRel = {...rel};
        if (axis === 'x') {
          newRel.y = rel.y * Math.cos(rad) - rel.z * Math.sin(rad);
          newRel.z = rel.y * Math.sin(rad) + rel.z * Math.cos(rad);
        } else if (axis === 'y') {
          newRel.x = rel.x * Math.cos(rad) - rel.z * Math.sin(rad);
          newRel.z = rel.x * Math.sin(rad) + rel.z * Math.cos(rad);
        } else if (axis === 'z') {
          newRel.x = rel.x * Math.cos(rad) - rel.y * Math.sin(rad);
          newRel.y = rel.x * Math.sin(rad) + rel.y * Math.cos(rad);
        }

        // Compute new world position
        let newPos = {
          x: center[0] + Math.round(newRel.x),
          y: center[1] + Math.round(newRel.y),
          z: center[2] + Math.round(newRel.z)
        };

        // Animate position
        cubelet.setAttribute('animation__pos', {
          property: 'position',
          to: `${newPos.x} ${newPos.y} ${newPos.z}`,
          dur: 600,
          easing: 'easeInOutQuad'
        });

        // Animate rotation (just axis for visual)
        let newRot = {...rot};
        if (axis === 'x') {
          newRot.x += angle;
        } else if (axis === 'y') {
          newRot.y += angle;
        } else if (axis === 'z') {
          newRot.z += angle;
        }
        cubelet.setAttribute('animation__rot', {
          property: 'rotation',
          to: `${newRot.x} ${newRot.y} ${newRot.z}`,
          dur: 600,
          easing: 'easeInOutQuad'
        });

        // Update stored position/rotation after animation completes
        setTimeout(() => {
          cubelet.setAttribute('position', `${newPos.x} ${newPos.y} ${newPos.z}`);
          cubelet.setAttribute('rotation', `${newRot.x} ${newRot.y} ${newRot.z}`);
          cubelet.setAttribute('data-x', newPos.x);
          cubelet.setAttribute('data-y', newPos.y);
          cubelet.setAttribute('data-z', newPos.z);
          // Remove animation so further clicks work
          cubelet.removeAttribute('animation__pos');
          cubelet.removeAttribute('animation__rot');
        }, 650);
      });
    }
    window.rotateSlice = rotateSlice;
    </script>
  </body>
</html>
