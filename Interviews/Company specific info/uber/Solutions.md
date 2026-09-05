

---

### In memory file system 

```cpp
class Node{
    public:
    unordered_map<string,Node*> children;
    Node* parent;
    string name;
};

class FileSystem{
    private:

    Node* root;
    unordered_map<Node*,string> values;

    string curr_path;
    Node* curr_working_dir;

    public:
    FileSystem(){
        root = new Node();
        curr_working_dir=root;
        curr_path="/";
        root->name="/";
    }

    vector<string> segregate(string s){
        int n = s.size();
        vector<string> splitted_path;
        string temp;
        for(int i=0;i<n;i++){
            if(s[i]=='/'){
                if(temp.size()>0){
                    splitted_path.push_back(temp);
                }
                temp="";
            }else{
                temp+=s[i];
                if(i==n-1){
                    splitted_path.push_back(temp);
                }
            }
        }
        
        return splitted_path;
    }

    bool does_path_exists(string path,bool exclude_end=false){
        auto splitted_path=segregate(path);
        Node* curr = root;
        int end_index = splitted_path.size();
        if(exclude_end)end_index-=1;

        for(int i=0;i<end_index;i++){
            if(curr->children.find(splitted_path[i])==curr->children.end()){
                return false;
            }
            curr = curr->children[splitted_path[i]];
        }
        return true;
    }

    void mkdir(Node* node,string s){
        auto splitted_path = segregate(s);
        Node* curr = root;
        for(auto path:splitted_path){
            if(curr->children.find(path)==curr->children.end()){
                curr->children[path]=new Node();
                curr->children[path]->parent=curr;
            }
            curr->name=path;
            curr=curr->children[path];
        }
    }

    void mkdir(string s){
        mkdir(root,s);
    }



    void addContentToFile(string path,string content){
        
        if(!does_path_exists(path,true)){
            throw runtime_error("Path does not exists");
        }

        auto splitted_path=segregate(path);
        Node* curr = root;
        int end_index = splitted_path.size();

        for(int i=0;i<end_index-1;i++){
            curr = curr->children[splitted_path[i]];
        }
        curr->children[splitted_path.back()]=new Node();
        values[curr->children[splitted_path.back()]]=content;
    }

    string readcontentFromFile(string path){
        
        if(!does_path_exists(path,false)){
            throw runtime_error("Path does not exist");
        }
        
        auto splitted_path = segregate(path);

        Node* curr = root;

        for(auto ele:splitted_path){
            curr=curr->children[ele];
        }
        return values[curr];
    }

    vector<string> ls(string path){
        Node* curr=root;

        if(!does_path_exists(path,false)){
            throw runtime_error("Path does not exist");
        }

        auto splitted_path=segregate(path);

        for(auto ele:splitted_path){
            curr=curr->children[ele];
        }
        vector<string> dir_ele;

        for(auto ele:curr->children){
            dir_ele.push_back(ele.first);
        }
        return dir_ele;
    }

    void cd(string path){
        auto splitted_path = segregate(path);
        Node* curr = (path[0]=='/'?root:curr_working_dir);
        for(auto ele:splitted_path){
            if(ele==".."){
                if(!(curr->parent)){
                    throw runtime_error("Path is invalid");
                }
                curr = curr->parent;
            }else if(ele=="."){
                continue;
            }else{
                if(curr->children.find(ele)==curr->children.end()){
                    throw runtime_error("Path is invalid");
                }
                curr = curr->children[ele];
            }
        }
        curr_working_dir=curr;
    }

    string pwd() {
        if (curr_working_dir == root) return "/";

        string res = "";
        Node* temp = curr_working_dir;
        
        while (temp != root) {
            res = "/" + temp->name + res;
            temp = temp->parent;
        }
        return res;
    }
};

```

### Freq counter 

```cpp
class KeyCounter{
    private:
    queue<pair<int,int>> q;
    unordered_map<int,int> counts;
    int ttl;
    int tot=0;

    KeyCounter(int t):ttl(t){}


    void remove_stale(int t){
        while(!q.empty() && q.front().first<t){
            int key=q.front().second;
            counts[key]--;
            if(counts[key] == 0){
                counts.erase(key);
            }
            q.pop();
            tot-=1;
        }
    }

    public:

    void put_element(int t,int key){
        remove_stale(t);
        counts[key]++;
        q.push({t+ttl,key});
        tot+=1;
    }

    int get_element_count(int t,int key){
        remove_stale(t);
        return counts[key];
    }

    int get_total(int t){
        remove_stale(t);
        return tot;
    }

};

```