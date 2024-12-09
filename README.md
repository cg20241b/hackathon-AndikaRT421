[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/_wHFcbvB)
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=17481536)

<hr>

# 🌟 Three.js Shader Hackathon Project 🌟

> **👤 Name:** Andika Rahman Teja  
> **🆔 NRP:** 5025221022

---

## 🖥️ Live Demo  
🎥 Check out the project in action!  

![Demo](/assets/demo_grafkom.gif)

---

## 🎯 **Project Goal**

This project demonstrates custom vertex and fragment shaders using Three.js in a GitHub Codespace environment. The goal is to gain practical experience in shader-based materials, lighting, and object manipulation, culminating in an interactive 3D scene rendered in the browser.

---

## 🛠️ **Instruction**

1. **Project Setup:**
   - Clone this repository into a GitHub Codespace.
   - Install all required dependencies (e.g., `npm install`) and run the development server.
   - Ensure that the project runs correctly and can be previewed directly within the Codespace environment.

> 🚀 How to run the project:

```sh
npm install
http-server
```

2. 🔍 **Scene Composition:**
   - Determine the last alphabet character of your name. For example, if your name is "Ardi," the last alphabet is "i."
   - Determine the last digit of your student ID. For example, if your student ID is `123456`, the last digit is `6`.
   - Create 3D text meshes representing:
     - The alphabet character (placed on the **left** side of the view).
     - The digit (placed on the **right** side of the view).

> **Example:**
> - Name: Andika => Last Alphabet: `A`
> - NRP: 5025221022 => Last Digit: `2`

3. 🎨 **Color Specification:**
   - Assign the alphabet mesh your favorite color from Assignment 1.
   - Assign the digit mesh the complementary color of the alphabet’s color.

> **Example:**
> - Alphabet Color: Old Gold (#D0B341) or RGB(208, 179, 65)
> - Digit Color: Warm Blue (#415ED0) or RGB(65, 94, 208)

4. 🧊 **Central Cube (Light Source):**
   - Add a small cube at the center of the scene, between the alphabet and the digit meshes.
   - The cube should appear to "glow" white, serving as a point light source.
   - Use a dedicated `ShaderMaterial` that simulates light emission.

> **Cube Specifications:**

```js
const cubeGeometry = new THREE.BoxGeometry(0.3, 0.3, 0.1);
const glowMaterial = new THREE.ShaderMaterial({
uniforms: { time: { value: 0.0 } },
vertexShader: `void main() {
   gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}`,
fragmentShader: `uniform float time;
void main() {
   float speed = abs(sin(time * 0.01)) * 2.5 + 1.0;
   float intensity = abs(sin(time * speed)) * 0.4 + 0.4;
   gl_FragColor = vec4(vec3(1.0) * intensity, 1.0);
}`,
});
const cube = new THREE.Mesh(cubeGeometry, glowMaterial);
cube.position.set(0, 0, 0);
scene.add(cube);

pointLight.position.copy(cube.position);
```

5. 🔠**Shader Materials for Characters:**
   Two distinct `ShaderMaterial` instances must be created—one for the alphabet and one for the digit—featuring custom shading models influenced by the cube’s position as a point light source.

   - 🅰️**Alphabet ShaderMaterial:**
     - **Ambient:** Use an intensity of `0.abc`, where `abc = (last three digits of your Student ID) + 200`. For example, if the last three digits are `456`, then `abc = 656` and ambient intensity = `0.656`.
     - **Diffuse:** Implement diffuse lighting influenced by the cube’s position.
     - **Specular (Plastic):** Add a plastic-like specular highlight, using a Blinn-Phong or Phong model with moderate shininess.

   - 2️⃣**Digit ShaderMaterial:**
     - **Ambient:** Apply the same ambient intensity calculation as above.
     - **Diffuse:** Diffuse lighting also comes from the cube’s point light source.
     - **Specular (Metal):** Add a metal-like specular highlight. Make the specular color more closely related to the base color and increase the reflectivity or shininess for a metallic appearance.

> **Example Specular (plastic):**

```js
const createAlphabetMaterial = (color) => {
   return new THREE.ShaderMaterial({
      uniforms: {
      cubePosition: { value: new THREE.Vector3(0, 0, 0) },
      },
      vertexShader: `
      varying vec3 vPosition;
      void main() {
         vPosition = (modelMatrix * vec4(position, 1.0)).xyz;
         gl_Position = projectionMatrix * viewMatrix * vec4(vPosition, 1.0);
      }
      `,
      fragmentShader: `
         uniform vec3 cubePosition;
         uniform vec3 lightPosition;
         uniform vec3 viewPosition;
         varying vec3 vPosition;
         void main() {
         vec3 fragPosition = vPosition;
         vec3 normal = normalize(abs(fragPosition - cubePosition));

         vec3 lightDir = normalize(lightPosition - fragPosition);
         vec3 viewDir = normalize(viewPosition - fragPosition);

         // Ambient component
         vec3 ambient = 0.222 * vec3(${color});

         // Diffuse component
         float diff = max(dot(normal, lightDir), 0.0);
         vec3 diffuse = diff * vec3(${color});

         // Specular component
         vec3 reflectDir = reflect(-lightDir, normal);
         float spec = pow(max(dot(viewDir, reflectDir), 0.0), 10.0);
         vec3 specular = vec3(1.0) * spec;

         // Combine components
         vec3 color = ambient + diffuse + specular;
         gl_FragColor = vec4(color, 1.0);
         }
      `,
   });
};
```

> **Example Specular (metal):**

```js
const createDigitMaterial = (color) => {
   return new THREE.ShaderMaterial({
   uniforms: {
      cubePosition: { value: new THREE.Vector3(0, 0, 0) },
   },
   vertexShader: `
      varying vec3 vPosition;
      void main() {
         vPosition = (modelMatrix * vec4(position, 1.0)).xyz;
         gl_Position = projectionMatrix * viewMatrix * vec4(vPosition, 1.0);
      }
   `,
   fragmentShader: `
         uniform vec3 cubePosition;
         uniform vec3 lightPosition;
         uniform vec3 viewPosition;
         varying vec3 vPosition;
         void main() {
         vec3 fragPosition = vPosition;
         vec3 normal = normalize(abs(fragPosition - cubePosition));

         vec3 lightDir = normalize(lightPosition - fragPosition);
         vec3 viewDir = normalize(viewPosition - fragPosition);

         // Ambient component
         vec3 ambient = 0.222 * vec3(${color});

         // Diffuse component
         float diff = max(dot(normal, lightDir), 0.0);
         vec3 diffuse = diff * vec3(${color});

         // Specular component
         vec3 reflectDir = reflect(-lightDir, normal);
         float spec = pow(max(dot(viewDir, reflectDir), 0.0), 100.0);
         vec3 specular = vec3(${color}) * spec;

         // Combine components
         vec3 color = ambient + diffuse + specular;
         gl_FragColor = vec4(color, 1.0);
         }
   `,
   });
};
```

🔑 **Key Differences:**
> - The `specular` calculation for the **plastic** material uses a lower exponent (e.g., 10.0) for a broader highlight.
> - The `specular` calculation for the **metal** material uses a higher exponent (e.g., 100.0) for a tighter, more reflective highlight.

6. 🕹️ **Interactivity:**
   - 🎲 **Cube Movement:**  
     - Press `W` to move the cube upward (increase Y coordinate).
     - Press `S` to move the cube downward (decrease Y coordinate).
   
   - 📽️ **Camera Movement:**  
     - Press `A` to move the camera left.
     - Press `D` to move the camera right.
     The camera should translate linearly, not orbit around the objects.

> **Example Cube Movement:**

```js
window.addEventListener("keydown", (event) => {
const step = 0.1;
switch (event.key) {
   case "w": cube.position.y += step; break;
   case "s": cube.position.y -= step; break;
   case "ArrowLeft": cube.position.x -= step; break;
   case "ArrowRight": cube.position.x += step; break;
   case "a": camera.position.x += step; break;
   case "d": camera.position.x -= step; break;
   case "ArrowUp": camera.position.y -= step; break;
   case "ArrowDown": camera.position.y += step; break;
}
});
```

7. 🎛️ **Version Control & Submission:**
   - Commit changes regularly to this repository.
   - Push the final version before the submission deadline.

🕰️ **Deadline:**  
**Monday, 9 December 2021, 21:00**

---

📝 **Additional Notes:**
- Ensure that all shader code is well-documented and commented.
- Test the interactivity (W, S, A, D keys) to confirm the correct behavior.
- Verify that the ambient, diffuse, and specular components are visually distinguishable and aligned with the specified materials (plastic vs. metal appearance).

After completing all requirements, your final project should showcase a properly lit scene with custom shader materials, a dynamic light source, and interactive controls for both the light and camera.
