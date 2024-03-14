## TransferContract

    class TransferContract(Model):
        date_create = models.DateTimeField(auto_now_add=True)
        date_updated = models.DateTimeField(auto_now=True)
        status = models.CharField(max_length=128, choices=[
            ['approved', 'Approved'],
            ['rejected', 'Rejected'],
            ['waiting', 'Waiting'],
            ['used', 'Used']
        ], default='waiting')
        child = models.ForeignKey(to='Child', on_delete=models.PROTECT)
        old_owner = models.ForeignKey(to='customusers.CustomUser', on_delete=models.PROTECT, related_name='old_contacts')
        new_owner = models.ForeignKey(to='customusers.CustomUser', on_delete=models.PROTECT, related_name='transfered_contacts')

## Сигналы
**post_save**
Если смена статуса происходит с ожидания на подтверждено
Меняем у ребенка ответственного и сохраняем. Это вызовет **C3** с обнаружением записи.
После чего меняем статус этой записи с НЕ использованной на использованную.
Повторная смена будет недоступна по этой записи.