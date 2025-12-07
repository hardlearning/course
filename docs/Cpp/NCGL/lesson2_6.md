# 6 使用vector传递和获取内存

```cpp
#include <iostream>
#include <vector>
using namespace std;

class XData {
public:
    XData() {
        cout << "Create XData" << endl;
    }
    XData(const XData& d) {
        this->index = d.index;
        cout << "Copy XData " << index << endl;
    }
    XData& operator=(const XData& d) {
        this->index = d.index;
        cout << "=XData " << index << endl;
        return *this;
    }
    ~XData() {
        cout << "Drop XData" << endl;
    }

    int index = 0;
};

vector<XData> TestVec(vector<XData> &data) {
    cout << "data.data() = " << data.data() << endl;
    vector<XData> re;
    re.resize(3);
    cout << "re.data() = " << re.data() << endl;
    cout << "re Byte = " << re.size() * sizeof(XData) << endl;
    return re;
}

int main() {
    // 使用vector传递和获取内存
    // 功能等同于string
    // 可用于传递不同的类型
    // 可以用作返回值 支持move
    {
        vector<XData> vd;
        XData d;
        d.index = 1;
        vd.push_back(d);
        d.index = 2;
        vd.push_back(d);
        d.index = 3;
        vd.push_back(d);
        cout << "vd.data() = " << vd.data() << endl;
        auto rd = TestVec(vd);
        cout << "rd.data() = " << rd.data() << endl;
    }
    return 0;
}
```
