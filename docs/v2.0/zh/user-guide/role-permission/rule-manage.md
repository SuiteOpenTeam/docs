# 角色管理
SuiteBoot的权限体系基于角色来控制。

菜单路径：系统设置-用户和权限-角色管理

![](https://tcs-devops.aliyuncs.com/storage/1134f38de4a7004500f19b0b3ea690727765?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTcxODA5NjEzMiwiaWF0IjoxNzE3NDkxMzMyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMzRmMzhkZTRhNzAwNDUwMGYxOWIwYjNlYTY5MDcyNzc2NSJ9.QjAOXusDhcHci18h0iUvk6cXt0ZZ1PRcs3oauOvF27c&download=image.png "")

在该菜单下管理角色信息，新建/更新/删除角色以及角色下的用户列表。

每个角色可以分配给多个用户，每个用户也可以绑定多个角色。

当用户登录时，该用户具有的权限为其绑定的所有角色的权限的并集。

例如，用户具有A，B两个角色，对于【仓库】对象，

- A角色对仓库有新建、更新操作权限；

- B角色对仓库有删除、禁用操作权限；

则该用户对【仓库】对象有新建、更新、删除、禁用的操作权限。



对于数据权限，则取最大值，例如

- A角色对隶属本部门及下级部门的仓库有权限
- B角色对所有仓库有权限

则该用户对【所有】仓库有数据权限。