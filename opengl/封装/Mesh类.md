`Mesh.h`
```cpp
struct Vertex {  
    glm::vec3 position;  
    glm::vec3 normal;  
};  
  
class Mesh {  
    unsigned int VAO = 0, VBO = 0, EBO = 0;  
  
    void setupMesh();  
  
public:  
    std::vector<Vertex> vertices;  
    std::vector<unsigned int> indices;  
    glm::vec3 color;  
  
    Mesh(const std::vector<Vertex> &vertices, const std::vector<unsigned int> &indices,  
         glm::vec3 color = glm::vec3(1.0f));  
  
    void Draw(const Shader &shader) const;  
};
```
`Mesh.cpp`
```cpp
Mesh::Mesh(const std::vector<Vertex> &vertices, const std::vector<unsigned int> &indices, const glm::vec3 color) {  
    this->vertices = vertices;  
    this->indices = indices;  
    this->color = color;  
  
    setupMesh();  
}  
  
void Mesh::setupMesh() {  
    if (vertices.empty() || indices.empty()) {  
        return;  
    }  
  
    glGenVertexArrays(1, &VAO);  
    glGenBuffers(1, &VBO);  
    glGenBuffers(1, &EBO);  
  
    glBindVertexArray(VAO);  
    glBindBuffer(GL_ARRAY_BUFFER, VBO);  
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(Vertex), vertices.data(), GL_STATIC_DRAW);  
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);  
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size() * sizeof(unsigned int), indices.data(), GL_STATIC_DRAW);  
  
    glEnableVertexAttribArray(0);  
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *) 0);  
    glEnableVertexAttribArray(1);  
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *) offsetof(Vertex, normal));  
  
    glBindVertexArray(0);  
}  
  
void Mesh::Draw(const Shader &shader) const {  
    if (VAO == 0 || indices.empty()) {  
        return;  
    }  
  
    shader.setVec3("u_Color", color);  
  
    glBindVertexArray(VAO);  
    glDrawElements(GL_TRIANGLES, static_cast<GLsizei>(indices.size()), GL_UNSIGNED_INT, nullptr);  
    glBindVertexArray(0);  
}
```