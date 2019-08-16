# dynamic-permission
动态加载菜单权限-vue


根据当前登录的用户的角色，来分配不同的使用权限。比如：
  A用户，有'首页'、'销售中心'的权限
  B用户，有'首页'、'产品中心'的权限
  C用户，有所有模块的权限

`所有用户都必须拥有 '首页' 的权限`，因为整个项目的路由解构，是放在dashboard下面的。如果想某些角色的用户不能拥有'首页'的权限，这时需要整体调整 路由 的解构。

目前项目中的菜单权限是`单角色`的。

### 本项目中动态加载菜单权限的原理
  给每一个路由一个 `唯一的resCode` (和后台的resCode对应)，然后根据permissions接口返回的数据，遍历路由，加载出当前用户拥有的权限的菜单。

### 主要code
目录：src/store/modules/permission

```
  /**
 * 过滤账户是否拥有某一个权限，并将菜单从加载列表移除
 *
 * @param roles
 * @param route
 * @returns {boolean}
 */
function hasPermission (roles, route) {
  for (let i = 0, len = roles.permissionListParent.length; i < len; i++) {
    if (roles.permissions[i].resCode === route.meta.resCode) {
      route.meta.permission = roles.permissions[i].permissionId
    }
    if (roles.permissions[i].children) {
      roles.permissions[i].children.forEach(item => {
        if (item.resCode === route.meta.resCode) {
          route.meta.permission = item.permissionId
        }
      })
    }
  }

  if (route.meta) {
    let flag = false
    let childFlag = false
    for (let m = 0, len = roles.permissions.length; m < len; m++) {
      flag = route.meta.permission.includes(roles.permissions[m].permissionId)

      if (roles.permissions[m].children) {
        for (const item of roles.permissions[m].children) {
          childFlag = route.meta.permission.includes(item.permissionId)
          if (childFlag) {
            return true
          }
        }
      }

      if (flag && childFlag) {
        return true
      }

      if (flag) {
        return true
      }
    }
    return false
  }
  // return true
}
```
