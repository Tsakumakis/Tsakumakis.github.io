<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Rubik's Cube Slice Rotation - Single Cubelet Version</title>
    <script src="https://aframe.io/releases/1.7.0/aframe.min.js"></script>
    <!-- Add orbit-controls component -->
    <script src="https://unpkg.com/aframe-orbit-controls@1.2.0/dist/aframe-orbit-controls.min.js"></script>
    <style>
        #fun-menu {
          position: bottom;
          top: 10px;
          right: 10px;
          z-index: 10;
          background: rgba(255,255,255,0.85);
          padding: 12px;
          border-radius:8px;
          font-family: sans-serif;
        }
        #fun-menu button { margin: 2px 4px; padding: 6px 12px; border-radius: 5px; border: none; background: #ff6a95; color: white; font-weight: bold; cursor: pointer;}
        #fun-menu button:hover { background: #b54562;}
    </style>
      <div id="fun-menu">
      
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
    </body>
  </head>
  <body>
    <div id="rubiks-menu">
      <b>Rotate Slice (grouped, centered):</b><br>
      <button onclick="rotateSlice('U')">U (Top, y=3)</button>
      <button onclick="rotateSlice('D')">D (Bottom, y=1)</button>
      <button onclick="rotateSlice('L')">L (Left, x=-1)</button>
      <button onclick="rotateSlice('M')">M (middle y=0)</button>
      <button onclick="rotateSlice('R')">R (Right, x=1)</button>
      <button onclick="rotateSlice('F')">F (Front, z=-2)</button>
      <button onclick="rotateSlice('B')">B (Back, z=-4)</button>
    </div>
    
    <!-- SINGLE SCENE with camera added -->
    <a-scene>
      <!-- Camera with orbit controls ONLY -->
      <a-entity
        id="camera"
        camera="fov: 50; near: 0.1; far: 100"
        position="4 5 8"
        orbit-controls="
          target: 0 2 -3;
          enableDamping: true;
          dampingFactor: 0.05;
          rotateSpeed: 0.5;
          zoomSpeed: 0.5;
          minDistance: 3;
          maxDistance: 15;
          enablePan: true;
          enableZoom: true;
          enableRotate: true;
          screenSpacePanning: true"
      ></a-entity>
      
      <a-assets>
        <a-asset-item id="cube-model" src="https://kostasgian21.github.io/computer_graphics/rubiks_cube_standard_solid_v3.glb"></a-asset-item>
        <img id="skyTexture" src="https://res.cloudinary.com/myhq/image/upload/w_1920/f_auto/web/ssr/common/whymyhq/new-banner/managed-office.png" crossorigin="anonymous"/>
      </a-assets>
      <a-sky src="#skyTexture"></a-sky>
      <a-entity light="type: directional; intensity:6.2; color:#fff" position="1 4 2"></a-entity>
      <a-entity light="type: ambient; intensity:2.35; color:#fff"></a-entity>
      <a-entity light="type: point; intensity:3.6; distance:10" position="-3 2 0"></a-entity>
      <a-entity light="type: point; intensity:3.6; distance:10" position="0 1 -5"></a-entity>
      <a-entity id="cube-cluster"></a-entity>
      <a-plane position="0 0 -4" rotation="-90 0 0" width="4" height="4" color="#7BC8A4"></a-plane>
    </a-scene>

    <script>
    // Cube parameters
    const positions = [-1, 0, 1];
    const layersY = [1, 2, 3];
    const layersZ = [-2, -3, -4];
    const cubeSize = 0.8;
    const model = '#cube-model';

    // Slices definitions, with their centers and match functions
    const sliceDefs = {
      U:  {center: [0,3,-3],  match: (x,y,z) => y===3, axis: 'y', angle: 90},      // Top row
      D:  {center: [0,1,-3],  match: (x,y,z) => y===1, axis: 'y', angle: -90},     // Bottom row
      L:  {center: [-1,2,-3], match: (x,y,z) => x===-1, axis: 'x', angle: -90},    // Left column
      M:  {center: [0,2,-3],  match: (x,y,z) => x===0, axis: 'x', angle: 90},      // Middle column
      R:  {center: [1,2,-3],  match: (x,y,z) => x===1, axis: 'x', angle: 90},      // Right column
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
    
    // reset cube to initial state
    function resetCube(){
      cubelets.forEach(cubelet => {
        const x = cubelet.getAttribute('data-x');
        const y = cubelet.getAttribute('data-y');
        const z = cubelet.getAttribute('data-z');
        cubelet.setAttribute('position', `${x} ${y} ${z}`);
        cubelet.setAttribute('rotation', `0 0 0`);
      })
    }

    // Auto shuffle toggle: keeps applying random moves until stopped
    let _autoShuffleId = null;
    function toggleAutoShuffle() {
      if (_autoShuffleId != null) {
        clearInterval(_autoShuffleId);
        _autoShuffleId = null;
        return;
      }
      _autoShuffleId = setInterval(() => {
        const faces = Object.keys(sliceDefs);
        const face = faces[Math.floor(Math.random() * faces.length)];
        rotateSlice(face);
      }, 700);
    }

    window.resetCube = resetCube;
    window.toggleAutoShuffle = toggleAutoShuffle;
    </script>
    
    <!-- Bottom menu: reset + two extra buttons -->
    <style>
      #bottom-menu {
        position: absolute;
        bottom: 18px;
        left: 50%;
        transform: translateX(-50%);
        z-index: 11;
        background: rgba(0,0,0,0.6);
        color: white;
        padding: 8px 12px;
        border-radius: 8px;
        font-family: sans-serif;
      }
      #bottom-menu button { margin: 0 6px; padding: 6px 10px; border-radius: 5px; border: none; background: #333; color: white; cursor: pointer; }
      #bottom-menu button:hover { background: #555; }
    </style>
    
    <div id="bottom-menu">
      <button onclick="resetCube()">Reset Cube</button>
      <button onclick="toggleAutoShuffle()">Auto Shuffle</button>
    </div>
    
  </body>
</html>
