**自关联地区Model**
```python
class AreaModel(models.Model):
    """
    行政区划
    null=True：（必写项）允许省级的父级为空
    blank=True：（必写项）约束将来在admin站点
    parent表单里可以不填
    on_delete：删除守护，比如将来需要删除某个市，如果
    不做守护，会把这个市下面的区全都删掉
    """
    name = models.CharField(max_length=20, verbose_name='名称')
    parent = models.ForeignKey('self', on_delete=models.SET_NULL, related_name='subs', null=True, blank=True, verbose_name='上一级行政区')

    class Meta:
        db_table = 'CRM_Areas'
        verbose_name = '行政区划'
        verbose_name_plural = '行政区划'

    def __str__(self):
        return self.name
```
**地区序列化**
```python
class AreaSerializer(serializers.ModelSerializer):
    """
    行政区划信息序列化器
    """

    class Meta:
        model = AreaModel
        fields = ('id', 'name')


class SubAreaSerializer(serializers.ModelSerializer):
    """
    子行政区划信息序列化器
    """
    subs = AreaSerializer(many=True, read_only=True)

    class Meta:
        model = AreaModel
        fields = ('id', 'name', 'subs')
```
**ViewSet**
```python
class AreasViewSet(CacheResponseMixin, ReadOnlyModelViewSet):
    """
    行政区划信息
    """
    pagination_class = None  # 区划信息不分页

    def get_queryset(self):
        """
        提供数据集
        """
        if self.action == 'list':
            return AreaModel.objects.filter(parent=None)
        else:
            return AreaModel.objects.all()

    def get_serializer_class(self):
        """
        提供序列化器
        """
        if self.action == 'list':
            return AreaSerializer
        else:
            return SubAreaSerializer
```
数据来源: [http://www.tcmap.com.cn/list/jiancheng_list.html](http://www.tcmap.com.cn/list/jiancheng_list.html)