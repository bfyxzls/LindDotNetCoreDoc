### redis����ܽ�
1. ��key��ȡvalue�������Ҫ����value����ֻ�ܰ���ת��Ϊkey�������ٴ洢һ��k/v
1. ��key��ȡvalue��ʱ�临�Ӷ�Ϊo(1)�����ֵ���ͬ
1. �ֲ��������������ĳ��redis���ﵽ�Ŷӵ�Ч��
1. session������Ӧ��ʵ�ָ��ؾ���ʱ������session
1. ������������ͨ���Է�����aop���أ�ʵ�ֻ�����Ե�ע��
#### �ֲ�������
```
//����������ʱ��100ms,���������ǰִ����,�������ͷ���
if (redisManager.Instance.GetDatabase().LockTake("����", "ֵûʲô��", TimeSpan.FromMilliseconds(100)))
{
    try
    {
        Console.WriteLine("���ڴ�����");
        Thread.Sleep(1000);
    }
    catch (Exception)
    {
        throw;
    }
    finally
    {
        //����������ͷ�redis������,����Ҫ����100����
        redisManager.Instance.GetDatabase().LockRelease("����", "ֵûʲô��");
    }
}
```