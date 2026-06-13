<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Trái Đất Chi Tiết Cao - Núi, Sông, Nhà Cửa</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
    #info {
      position: absolute;
      top: 20px;
      left: 20px;
      color: white;
      background: rgba(0,0,0,0.7);
      padding: 12px 25px;
      border-radius: 40px;
      pointer-events: none;
      backdrop-filter: blur(10px);
      box-shadow: 0 4px 15px rgba(0,0,0,0.3);
      letter-spacing: 1px;
    }
    #controls {
      position: absolute;
      bottom: 30px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(20,20,30,0.8);
      backdrop-filter: blur(15px);
      color: #ddd;
      padding: 15px 30px;
      border-radius: 60px;
      display: flex;
      gap: 30px;
      font-size: 16px;
      box-shadow: 0 8px 25px rgba(0,0,0,0.5);
      border: 1px solid rgba(255,255,255,0.2);
      z-index: 10;
    }
    .btn {
      background: rgba(255,255,255,0.15);
      border: none;
      color: white;
      padding: 10px 25px;
      border-radius: 30px;
      font-size: 16px;
      cursor: pointer;
      transition: 0.3s;
      font-weight: bold;
      letter-spacing: 0.5px;
      backdrop-filter: blur(5px);
    }
    .btn:hover {
      background: rgba(255,255,255,0.35);
      box-shadow: 0 0 20px rgba(100,180,255,0.6);
    }
    .zoom-indicator {
      display: flex;
      align-items: center;
      gap: 10px;
      color: #aaddff;
    }
    @media (max-width: 700px) {
      #controls {
        flex-direction: column;
        gap: 10px;
        padding: 15px 25px;
        bottom: 20px;
      }
      #info {
        top: 10px;
        font-size: 14px;
        padding: 8px 20px;
      }
    }
  </style>
</head>
<body>
  <div id="info">
    <h2>🌍 TRÁI ĐẤT CHI TIẾT CAO</h2>
    <p>Zoom vào để thấy núi, sông, nhà cửa & cây cối</p>
  </div>
  <div id="controls">
    <button class="btn" id="resetView">↺ Xem toàn cảnh</button>
    <div class="zoom-indicator">
      <span>🔍</span> <span id="zoomLevel">Độ cao: 15000 km</span>
    </div>
    <button class="btn" id="toggleRotate">⏯️ Dừng quay</button>
  </div>

  <!-- Import maps và thư viện -->
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
      }
    }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
    import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

    // --- Khởi tạo scene, camera, renderer ---
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x050510);
    
    // Thêm các ngôi sao nền
    const starsGeometry = new THREE.BufferGeometry();
    const starsCount = 2000;
    const starsPositions = new Float32Array(starsCount * 3);
    for (let i = 0; i < starsCount * 3; i += 3) {
      starsPositions[i] = (Math.random() - 0.5) * 800;
      starsPositions[i+1] = (Math.random() - 0.5) * 500;
      starsPositions[i+2] = (Math.random() - 0.5) * 600;
    }
    starsGeometry.setAttribute('position', new THREE.BufferAttribute(starsPositions, 3));
    const starsMaterial = new THREE.PointsMaterial({color: 0xffffff, size: 0.35});
    const stars = new THREE.Points(starsGeometry, starsMaterial);
    scene.add(stars);

    // Camera
    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 5000);
    camera.position.set(0, 2, 18);
    camera.lookAt(0, 0, 0);

    // WebGL Renderer
    const renderer = new THREE.WebGLRenderer({ 
      antialias: true,
      powerPreference: "high-performance"
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    renderer.setClearColor(0x050510);
    document.body.appendChild(renderer.domElement);

    // CSS2 Renderer cho nhãn
    const labelRenderer = new CSS2DRenderer();
    labelRenderer.setSize(window.innerWidth, window.innerHeight);
    labelRenderer.domElement.style.position = 'absolute';
    labelRenderer.domElement.style.top = '0px';
    labelRenderer.domElement.style.left = '0px';
    labelRenderer.domElement.style.pointerEvents = 'none'; // cho phép click xuyên qua
    document.body.appendChild(labelRenderer.domElement);

    // Controls
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.rotateSpeed = 0.8;
    controls.zoomSpeed = 2.5;
    controls.minDistance = 2.2;
    controls.maxDistance = 50;
    controls.target.set(0, 0, 0);
    controls.update();

    // --- Ánh sáng ---
    // Ánh sáng mặt trời
    const sunLight = new THREE.DirectionalLight(0xfff5e6, 1.8);
    sunLight.position.set(50, 20, 40);
    sunLight.castShadow = true;
    sunLight.receiveShadow = true;
    sunLight.shadow.mapSize.width = 1024;
    sunLight.shadow.mapSize.height = 1024;
    sunLight.shadow.camera.near = 1;
    sunLight.shadow.camera.far = 150;
    sunLight.shadow.camera.left = -20;
    sunLight.shadow.camera.right = 20;
    sunLight.shadow.camera.top = 20;
    sunLight.shadow.camera.bottom = -20;
    scene.add(sunLight);
    
    const ambientLight = new THREE.AmbientLight(0x404066);
    scene.add(ambientLight);
    
    // Đèn fill nhẹ
    const fillLight = new THREE.DirectionalLight(0xccddff, 0.7);
    fillLight.position.set(-20, 5, -25);
    scene.add(fillLight);

    // --- Hành tinh chính: Trái Đất với texture chi tiết ---
    const earthGroup = new THREE.Group();
    scene.add(earthGroup);

    // Texture Trái Đất chất lượng cao (8K)
    const textureLoader = new THREE.TextureLoader();
    // Sử dụng texture miễn phí từ NASA/GIS (hoặc map 8k từ threejs examples)
    const earthMapUrl = 'https://threejs.org/examples/textures/planets/earth_atmos_2048.jpg';
    // Sử dụng bump map để tạo địa hình thô
    const earthBumpUrl = 'https://threejs.org/examples/textures/planets/earth_normal_2048.jpg';
    // Specular map cho đại dương
    const earthSpecularUrl = 'https://threejs.org/examples/textures/planets/earth_specular_2048.jpg';
    
    const earthGeometry = new THREE.SphereGeometry(5, 128, 128);
    const earthMaterial = new THREE.MeshPhongMaterial({
      map: textureLoader.load(earthMapUrl),
      bumpMap: textureLoader.load(earthBumpUrl),
      bumpScale: 0.15,
      specularMap: textureLoader.load(earthSpecularUrl),
      specular: new THREE.Color('grey'),
      shininess: 10,
    });
    
    const earth = new THREE.Mesh(earthGeometry, earthMaterial);
    earth.castShadow = true;
    earth.receiveShadow = true;
    earthGroup.add(earth);

    // --- Lớp mây (bán trong suốt) ---
    const cloudGeometry = new THREE.SphereGeometry(5.08, 128, 128);
    const cloudTexture = textureLoader.load('https://threejs.org/examples/textures/planets/earth_clouds_1024.png');
    const cloudMaterial = new THREE.MeshPhongMaterial({
      map: cloudTexture,
      transparent: true,
      opacity: 0.6,
      blending: THREE.AdditiveBlending,
      side: THREE.FrontSide
    });
    const clouds = new THREE.Mesh(cloudGeometry, cloudMaterial);
    earthGroup.add(clouds);

    // --- Hệ thống chi tiết bề mặt khi zoom gần (Procedural Details) ---
    // Chúng ta sẽ tạo một mặt phẳng chi tiết đại diện cho khu vực zoom vào.
    // Trong thực tế, đây là nơi bạn sẽ đặt các tile địa hình động.
    // Ở đây, tôi tạo sẵn một vùng đất chi tiết với nhà cửa, sông, núi nhỏ.
    
    const detailGroup = new THREE.Group();
    detailGroup.visible = false; // Ẩn cho đến khi zoom đủ gần
    earthGroup.add(detailGroup);
    
    // Vị trí đặt vùng chi tiết (trên bề mặt trái đất, vĩ độ/kinh độ ~ Châu Âu/Châu Á)
    // Sử dụng tọa độ cầu: radius = 5.02, lat ~ 45°, lon ~ 10°
    const lat = 45 * Math.PI / 180;
    const lon = 10 * Math.PI / 180;
    const detailRadius = 5.03;
    const detailPos = new THREE.Vector3(
      detailRadius * Math.cos(lat) * Math.cos(lon),
      detailRadius * Math.sin(lat),
      detailRadius * Math.cos(lat) * Math.sin(lon)
    );
    
    // Tạo một mặt phẳng cong nhỏ hoặc nhóm object đặt tại vị trí đó, hướng ra ngoài
    const surfaceNormal = detailPos.clone().normalize();
    const quaternion = new THREE.Quaternion().setFromUnitVectors(
      new THREE.Vector3(0, 0, 1),
      surfaceNormal
    );
    
    detailGroup.position.copy(detailPos);
    detailGroup.quaternion.copy(quaternion);
    
    // --- Xây dựng chi tiết địa hình trên mặt phẳng cục bộ (khoảng 1.2 x 1.2 đơn vị) ---
    function createDetailTile() {
      const tileGroup = new THREE.Group();
      
      // Nền đất (cỏ xanh, nâu)
      const groundGeometry = new THREE.CircleGeometry(0.65, 48);
      const groundMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x4caf50, 
        roughness: 0.8,
        side: THREE.DoubleSide
      });
      const ground = new THREE.Mesh(groundGeometry, groundMaterial);
      ground.rotation.x = -Math.PI/2;
      ground.position.z = 0;
      ground.receiveShadow = true;
      tileGroup.add(ground);
      
      // Thêm các mảng màu khác nhau cho đồng ruộng
      for (let i = 0; i < 12; i++) {
        const fieldGeo = new THREE.CircleGeometry(0.08 + Math.random()*0.1, 6);
        const fieldMat = new THREE.MeshStandardMaterial({ 
          color: new THREE.Color(`hsl(${50 + Math.random()*40}, 70%, 50%)`),
          roughness: 0.9
        });
        const field = new THREE.Mesh(fieldGeo, fieldMat);
        const angle = Math.random() * Math.PI * 2;
        const dist = 0.2 + Math.random() * 0.4;
        field.position.set(Math.cos(angle)*dist, Math.sin(angle)*dist, 0.01);
        field.rotation.x = -Math.PI/2;
        field.receiveShadow = true;
        tileGroup.add(field);
      }
      
      // --- Núi nhỏ (các khối nón) ---
      function createMountain(x, y, height, color = 0x8B7355) {
        const geom = new THREE.ConeGeometry(0.08, height, 8);
        const mat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.7 });
        const mountain = new THREE.Mesh(geom, mat);
        mountain.position.set(x, y, height/2);
        mountain.castShadow = true;
        mountain.receiveShadow = true;
        tileGroup.add(mountain);
        // Đỉnh tuyết
        if (height > 0.15) {
          const snowGeom = new THREE.ConeGeometry(0.04, height*0.2, 6);
          const snowMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.4 });
          const snow = new THREE.Mesh(snowGeom, snowMat);
          snow.position.set(x, y, height - 0.03);
          snow.castShadow = true;
          tileGroup.add(snow);
        }
      }
      
      // Dãy núi nhỏ
      createMountain(0.3, 0.25, 0.22, 0x7d6b5d);
      createMountain(0.38, 0.2, 0.18, 0x6b5a4b);
      createMountain(0.25, 0.32, 0.25, 0x8a7b6b);
      createMountain(-0.4, -0.3, 0.2, 0x6e5e4e);
      createMountain(-0.45, -0.25, 0.15, 0x7d6d5d);
      createMountain(0.5, -0.15, 0.12, 0x5e4e3e);
      
      // --- Sông ngòi (dải màu xanh uốn lượn) ---
      function createRiverPath(points, width = 0.04) {
        const curve = new THREE.CatmullRomCurve3(points);
        const tubeGeo = new THREE.TubeGeometry(curve, 64, width, 4, false);
        const tubeMat = new THREE.MeshStandardMaterial({ 
          color: 0x2196F3, 
          roughness: 0.2,
          emissive: new THREE.Color(0x001122),
          transparent: true,
          opacity: 0.9
        });
        const river = new THREE.Mesh(tubeGeo, tubeMat);
        river.position.z = 0.02;
        river.receiveShadow = true;
        tileGroup.add(river);
      }
      
      // Tạo sông chính
      const riverPoints1 = [
        new THREE.Vector3(-0.55, 0.45, 0),
        new THREE.Vector3(-0.35, 0.35, 0),
        new THREE.Vector3(-0.15, 0.4, 0),
        new THREE.Vector3(0.1, 0.3, 0),
        new THREE.Vector3(0.3, 0.15, 0),
        new THREE.Vector3(0.45, -0.1, 0),
        new THREE.Vector3(0.5, -0.35, 0),
        new THREE.Vector3(0.4, -0.55, 0)
      ];
      createRiverPath(riverPoints1, 0.045);
      
      const riverPoints2 = [
        new THREE.Vector3(-0.5, -0.4, 0),
        new THREE.Vector3(-0.3, -0.25, 0),
        new THREE.Vector3(-0.1, -0.15, 0),
        new THREE.Vector3(0.15, -0.2, 0),
        new THREE.Vector3(0.35, -0.35, 0)
      ];
      createRiverPath(riverPoints2, 0.035);
      
      // --- Nhà cửa (các khối hộp nhỏ) ---
      function createHouse(x, y, color = 0xcccccc, height = 0.06) {
        const group = new THREE.Group();
        // Thân nhà
        const bodyGeo = new THREE.BoxGeometry(0.045, 0.045, height);
        const bodyMat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.6 });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.position.z = height/2;
        body.castShadow = true;
        body.receiveShadow = true;
        group.add(body);
        // Mái nhà
        const roofGeo = new THREE.ConeGeometry(0.035, 0.03, 4);
        const roofMat = new THREE.MeshStandardMaterial({ color: 0xb55a3a, roughness: 0.7 });
        const roof = new THREE.Mesh(roofGeo, roofMat);
        roof.position.z = height + 0.01;
        roof.rotation.y = Math.PI/4;
        roof.castShadow = true;
        group.add(roof);
        group.position.set(x, y, 0);
        tileGroup.add(group);
      }
      
      // Khu dân cư nhỏ
      const housePositions = [
        [-0.25, 0.05], [-0.18, 0.12], [-0.3, 0.18], 
        [0.05, -0.25], [0.12, -0.18], [0.0, -0.32],
        [0.4, 0.35], [0.48, 0.28], [-0.42, -0.42],
        [-0.48, -0.35], [0.35, -0.48], [-0.35, -0.5]
      ];
      housePositions.forEach(pos => {
        createHouse(pos[0], pos[1], new THREE.Color(`hsl(${Math.random()*20 + 30}, 40%, 75%)`), 0.05 + Math.random()*0.04);
      });
      
      // --- Cây cối (hình nón nhỏ trên thân) ---
      function createTree(x, y) {
        const treeGroup = new THREE.Group();
        const trunkGeo = new THREE.CylinderGeometry(0.008, 0.01, 0.05);
        const trunkMat = new THREE.MeshStandardMaterial({ color: 0x8B5A2B });
        const trunk = new THREE.Mesh(trunkGeo, trunkMat);
        trunk.position.z = 0.025;
        trunk.castShadow = true;
        trunk.receiveShadow = true;
        treeGroup.add(trunk);
        
        const foliageGeo = new THREE.ConeGeometry(0.025, 0.045, 6);
        const foliageMat = new THREE.MeshStandardMaterial({ color: 0x2E7D32 });
        const foliage = new THREE.Mesh(foliageGeo, foliageMat);
        foliage.position.z = 0.06;
        foliage.castShadow = true;
        foliage.receiveShadow = true;
        treeGroup.add(foliage);
        
        treeGroup.position.set(x, y, 0);
        tileGroup.add(treeGroup);
      }
      
      // Rải cây
      for (let i = 0; i < 30; i++) {
        const angle = Math.random() * Math.PI * 2;
        const dist = 0.15 + Math.random() * 0.5;
        const tx = Math.cos(angle) * dist;
        const ty = Math.sin(angle) * dist;
        // Tránh đặt cây lên nhà hoặc sông (kiểm tra đơn giản)
        if (Math.abs(tx) < 0.5 && Math.abs(ty) < 0.5) {
          createTree(tx, ty);
        }
      }
      
      return tileGroup;
    }
    
    const detailTile = createDetailTile();
    detailGroup.add(detailTile);
    
    // Thêm một vài nhãn CSS cho thành phố (ví dụ)
    function addCityLabel(name, latDeg, lonDeg, color = '#ffffff') {
      const latR = latDeg * Math.PI/180;
      const lonR = lonDeg * Math.PI/180;
      const r = 5.15;
      const pos = new THREE.Vector3(
        r * Math.cos(latR) * Math.cos(lonR),
        r * Math.sin(latR),
        r * Math.cos(latR) * Math.sin(lonR)
      );
      const div = document.createElement('div');
      div.textContent = name;
      div.style.color = color;
      div.style.fontWeight = 'bold';
      div.style.fontSize = '14px';
      div.style.textShadow = '1px 1px 2px black';
      div.style.background = 'rgba(0,0,0,0.6)';
      div.style.padding = '2px 8px';
      div.style.borderRadius = '12px';
      const label = new CSS2DObject(div);
      label.position.copy(pos);
      earthGroup.add(label);
    }
    
    // Thêm một số thành phố nổi tiếng
    addCityLabel('Hà Nội', 21.03, 105.85, '#ffdd99');
    addCityLabel('Paris', 48.85, 2.35, '#ffccaa');
    addCityLabel('New York', 40.71, -74.00, '#aaccff');
    addCityLabel('Tokyo', 35.68, 139.76, '#ffaaaa');
    addCityLabel('Sydney', -33.86, 151.20, '#ccffcc');
    
    // --- Logic hiển thị chi tiết dựa trên khoảng cách camera ---
    function updateDetailVisibility() {
      const distanceToCenter = camera.position.distanceTo(new THREE.Vector3(0,0,0));
      // Khoảng cách đến tâm trái đất: nếu camera rất gần bề mặt (distance < 6.2) thì hiện chi tiết
      const threshold = 6.5;
      if (distanceToCenter < threshold && !detailGroup.visible) {
        detailGroup.visible = true;
        // Điều chỉnh vị trí detail group theo hướng camera? Giữ cố định hoặc dynamic.
        // Ở đây giữ cố định khu vực Châu Âu.
      } else if (distanceToCenter >= threshold && detailGroup.visible) {
        detailGroup.visible = false;
      }
      
      // Cập nhật chỉ báo zoom
      const altitude = (distanceToCenter - 5).toFixed(2);
      document.getElementById('zoomLevel').textContent = 
        distanceToCenter < threshold ? 
        `🔍 Độ cao: ${(altitude*1000).toFixed(0)} m (Chi tiết bề mặt)` : 
        `🌌 Độ cao: ${(altitude*1000).toFixed(0)} km`;
    }
    
    // --- Animation Loop ---
    let rotateEnabled = true;
    document.getElementById('toggleRotate').addEventListener('click', () => {
      rotateEnabled = !rotateEnabled;
      document.getElementById('toggleRotate').textContent = rotateEnabled ? '⏯️ Dừng quay' : '▶️ Quay';
    });
    
    document.getElementById('resetView').addEventListener('click', () => {
      camera.position.set(0, 2, 18);
      controls.target.set(0,0,0);
      controls.update();
    });
    
    function animate() {
      requestAnimationFrame(animate);
      
      // Xoay trái đất và mây
      if (rotateEnabled) {
        earthGroup.rotation.y += 0.0008;
        // Mây xoay nhanh hơn một chút
        clouds.rotation.y += 0.0004;
      }
      
      // Cập nhật sao nền xoay nhẹ
      stars.rotation.y += 0.0001;
      
      // Cập nhật hiển thị chi tiết
      updateDetailVisibility();
      
      // Cập nhật controls
      controls.update();
      
      // Render
      renderer.render(scene, camera);
      labelRenderer.render(scene, camera);
    }
    
    animate();
    
    // Xử lý resize
    window.addEventListener('resize', onWindowResize, false);
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
      labelRenderer.setSize(window.innerWidth, window.innerHeight);
    }
    
    console.log('🌐 Trái Đất chi tiết cao đã sẵn sàng! Zoom vào để khám phá núi, sông, nhà cửa.');
  </script>
</body>
</html>
