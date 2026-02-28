---
name: Creating Models
description: Step-by-step guide to create a new Odoo model with fields, constraints, and methods.
globs: ["models/**/*.py", "__manifest__.py"]
---

# Creating Odoo Models

## Steps

1. **Create the Python file** in `models/` directory (e.g. `models/my_model.py`).

2. **Define the model class**:
   ```python
   from odoo import models, fields, api, _
   from odoo.exceptions import ValidationError

   class MyModel(models.Model):
       _name = 'my.model'
       _description = 'My Model'
       _order = 'sequence, name'

       name = fields.Char(string='Name', required=True)
       sequence = fields.Integer(default=10)
       active = fields.Boolean(default=True)
       state = fields.Selection([
           ('draft', 'Draft'),
           ('confirmed', 'Confirmed'),
           ('done', 'Done'),
       ], default='draft', string='Status', tracking=True)
       partner_id = fields.Many2one('res.partner', string='Partner')
       line_ids = fields.One2many('my.model.line', 'model_id', string='Lines')
       tag_ids = fields.Many2many('my.model.tag', string='Tags')
       total = fields.Float(compute='_compute_total', store=True)

       @api.depends('line_ids.amount')
       def _compute_total(self):
           for record in self:
               record.total = sum(record.line_ids.mapped('amount'))

       @api.constrains('name')
       def _check_name(self):
           for record in self:
               if record.name and len(record.name) < 3:
                   raise ValidationError(_("Name must be at least 3 characters."))

       def action_confirm(self):
           self.write({'state': 'confirmed'})
   ```

3. **Register in `__init__.py`**: Add `from . import my_model` in `models/__init__.py`.

4. **Add to manifest**: Ensure `models/` is imported in the module's root `__init__.py`.

5. **Add security**: Create ACL in `security/ir.model.access.csv`.

6. **Create views**: Add form and list views in `views/my_model_views.xml`.

## Checklist
- [ ] `_name` and `_description` set
- [ ] Fields defined with proper types and attributes
- [ ] Computed fields have `@api.depends`
- [ ] Constraints use `@api.constrains`
- [ ] Model registered in `__init__.py` chain
- [ ] ACL entry in `ir.model.access.csv`
- [ ] Views created (at minimum form + list)
- [ ] Added to `__manifest__.py` data list
