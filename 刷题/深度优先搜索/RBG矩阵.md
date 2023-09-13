

```
#include<bits/stdc++.h>
using namespace std;

int n, m;
string s[1010];
bool vis1[1010][1010], vis2[1010][1010];
int mv[4][2] = {{0, 1} , {1, 0} , {0, -1} , {-1, 0}};

int dfs1(int x, int y) {
    if(x < 0 || x >= n) return 0;
    if(y < 0 || y >= m) return 0;
    if(vis1[x][y])     return 0;
    if(!vis1[x][y])    vis1[x][y] = true;
    for(int i = 0 ; i < 4 ; i++) {
        if(x + mv[i][0] < 0 || x + mv[i][0] >= n) continue;
        if(y + mv[i][1] < 0 || y + mv[i][1] >= m) continue;
        if(s[x + mv[i][0]][y + mv[i][1]] == s[x][y]) {
            dfs1(x + mv[i][0], y + mv[i][1]);
        }
    }
    return 1;
}

int dfs2(int x, int y) {
    if(x < 0 || x >= n) return 0;
    if(y < 0 || y >= m) return 0;
    if(vis2[x][y])     return 0;
    if(!vis2[x][y])    vis2[x][y] = true;
    for(int i = 0 ; i < 4 ; i++) {
        if(x + mv[i][0] < 0 || x + mv[i][0] >= n) continue;
        if(y + mv[i][1] < 0 || y + mv[i][1] >= m) continue;
        if(s[x][y] == 'G' || s[x][y] == 'B'){
            if(s[x + mv[i][0]][y + mv[i][1]] == 'G' || s[x + mv[i][0]][y + mv[i][1]] == 'B') {
                dfs2(x + mv[i][0], y + mv[i][1]);
            }
        } else if(s[x + mv[i][0]][y + mv[i][1]] == s[x][y]) {
            dfs2(x + mv[i][0], y + mv[i][1]);
        }
        
    }
    return 1;
}

int main(){
    int cnt = 0;
    cin >> n >> m;
    for(int i = 0 ; i < n ; i++) {
        cin >> s[i];
    }
    for(int i = 0 ; i < n ; i++) {
        for(int j = 0 ; j < m ; j++) {
            cnt += dfs1(i, j);
        }
    }
    for(int i = 0 ; i < n ; i++) {
        for(int j = 0 ; j < m ; j++) {
            cnt -= dfs2(i, j);
        }
    }

    cout << cnt << endl;
    return 0;
}
```

