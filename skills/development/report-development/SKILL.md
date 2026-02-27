---
name: Report Development
description: Create QWeb PDF reports and report actions in Odoo.
globs: ["report/**/*.xml", "report/**/*.py", "__manifest__.py"]
---

# Creating Reports

## Steps

### 1. Define Report Action (`report/my_model_report.xml`)
```xml
<odoo>
    <record id="action_report_my_model" model="ir.actions.report">
        <field name="name">My Model Report</field>
        <field name="model">my.model</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">my_module.report_my_model</field>
        <field name="report_file">my_module.report_my_model</field>
        <field name="print_report_name">'My Report - %s' % object.name</field>
        <field name="binding_model_id" ref="model_my_model"/>
        <field name="binding_type">report</field>
    </record>
</odoo>
```

### 2. Create QWeb Report Template (`report/my_model_report_template.xml`)
```xml
<odoo>
    <template id="report_my_model">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="doc">
                <t t-call="web.external_layout">
                    <div class="page">
                        <h2>
                            <span t-field="doc.name"/>
                        </h2>

                        <div class="row mt-4">
                            <div class="col-6">
                                <strong>Partner:</strong>
                                <span t-field="doc.partner_id"/>
                            </div>
                            <div class="col-6">
                                <strong>Date:</strong>
                                <span t-field="doc.create_date" t-options='{"widget": "date"}'/>
                            </div>
                        </div>

                        <table class="table table-sm mt-4">
                            <thead>
                                <tr>
                                    <th>Description</th>
                                    <th class="text-end">Amount</th>
                                </tr>
                            </thead>
                            <tbody>
                                <t t-foreach="doc.line_ids" t-as="line">
                                    <tr>
                                        <td><span t-field="line.name"/></td>
                                        <td class="text-end">
                                            <span t-field="line.amount"
                                                  t-options='{"widget": "monetary", "display_currency": doc.currency_id}'/>
                                        </td>
                                    </tr>
                                </t>
                            </tbody>
                            <tfoot>
                                <tr>
                                    <th>Total</th>
                                    <th class="text-end">
                                        <span t-field="doc.total"/>
                                    </th>
                                </tr>
                            </tfoot>
                        </table>
                    </div>
                </t>
            </t>
        </t>
    </template>
</odoo>
```

### 3. Custom Report Data (Optional - `report/my_model_report.py`)
```python
from odoo import api, models


class MyModelReport(models.AbstractModel):
    _name = 'report.my_module.report_my_model'
    _description = 'My Model Report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['my.model'].browse(docids)
        return {
            'doc_ids': docids,
            'doc_model': 'my.model',
            'docs': docs,
            'data': data,
            'extra_data': self._compute_extra_data(docs),
        }

    def _compute_extra_data(self, docs):
        return {'summary': f'{len(docs)} records'}
```

### 4. Update Manifest
```python
'data': [
    'report/my_model_report.xml',
    'report/my_model_report_template.xml',
],
```

## Checklist
- [ ] Report action uses `qweb-pdf` as `report_type`
- [ ] Template calls `web.html_container` and `web.external_layout`
- [ ] Template iterates over `docs` recordset
- [ ] Uses `t-field` for field rendering (handles formatting/translation)
- [ ] Report files listed in manifest
- [ ] Binding set on the action for the Print menu
- [ ] Custom report model named `report.<module>.<report_name>` (if needed)
