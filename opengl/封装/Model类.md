`Model.h`
```cpp
class Model {  
    std::vector<Mesh> meshes;  
    std::string directory;  
  
    void loadModel(const std::string &path);  
  
    void processNode(aiNode *node, const aiScene *scene);  
  
    Mesh processMesh(const aiMesh *mesh, const aiScene *scene);  
  
public:  
    Model(const std::string &path) { loadModel(path); }  
  
    void Draw(const Shader &shader) const;  
};
```
`Model.cpp`
```cpp
void Model::loadModel(const std::string &path) {  
    Assimp::Importer importer;  
    const aiScene *scene = importer.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs);  
    if (!scene || scene->mFlags & AI_SCENE_FLAGS_INCOMPLETE || !scene->mRootNode) {  
        std::cout << "ERROR::ASSIMP::" << importer.GetErrorString() << std::endl;  
        return;  
    }  
    const auto sep = path.find_last_of("/\\");  
    directory = (sep == std::string::npos) ? std::string() : path.substr(0, sep);  
    processNode(scene->mRootNode, scene);  
}  
  
void Model::processNode(aiNode *node, const aiScene *scene) {  
    for (unsigned int i = 0; i < node->mNumMeshes; i++) {  
        aiMesh *mesh = scene->mMeshes[node->mMeshes[i]];  
        meshes.push_back(processMesh(mesh, scene));  
    }  
  
    for (unsigned int i = 0; i < node->mNumChildren; i++) {  
        processNode(node->mChildren[i], scene);  
    }  
}  
  
Mesh Model::processMesh(const aiMesh *mesh, const aiScene *scene) {  
    std::vector<Vertex> vertices;  
    std::vector<unsigned int> indices;  
    glm::vec3 myColor = glm::vec3(0.2f, 0.5f, 0.9f);  
  
    for (unsigned int i = 0; i < mesh->mNumVertices; i++) {  
        Vertex vertex;  
        vertex.position = glm::vec3(mesh->mVertices[i].x, mesh->mVertices[i].y, mesh->mVertices[i].z);  
        if (mesh->HasNormals()) {  
            vertex.normal = glm::vec3(mesh->mNormals[i].x, mesh->mNormals[i].y, mesh->mNormals[i].z);  
        } else {  
            vertex.normal = glm::vec3(0.0f, 1.0f, 0.0f);  
        }  
        vertices.push_back(vertex);  
    }  
  
    for (unsigned int i = 0; i < mesh->mNumFaces; i++) {  
        aiFace face = mesh->mFaces[i];  
        for (unsigned int j = 0; j < face.mNumIndices; j++) {  
            indices.push_back(face.mIndices[j]);  
        }  
    }  
  
    return Mesh(vertices, indices, myColor);  
}  
  
void Model::Draw(const Shader &shader) const {  
    for (const auto &mesh: meshes) {  
        mesh.Draw(shader);  
    }  
}
```