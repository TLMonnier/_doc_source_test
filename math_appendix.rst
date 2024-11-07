
.. image:: images/banner_crop.png
   :width: 100 %
   :alt: Sponsors' logo banner

|

======================
Commuting choice model
======================

The terms highlighted in orange relate to functional assumptions needed to solve the model. For each mode :math:`m`, residential location :math:`x`, job center :math:`c`, income group :math:`i`, and worker :math:`j`, **expected commuting cost** is:

.. math::

    t_{mj}(x,c,w_{ic}) = \chi_i(\tau_m(x,c) + \delta_m(x,c)w_{ic}) + \textcolor{orange}{\epsilon_{mxcij}}

* :math:`w_{ic}` is the (calibrated) wage earned.
* :math:`\chi_i` is the employment rate of the household, composed of two agents (``household_size`` parameter).
* :math:`\tau_m(x,c)` is the monetary transport cost.
* :math:`\delta_m(x,c)` is a time opportunity cost parameter (the fraction of working time spent commuting).
* :math:`\textcolor{orange}{\epsilon_{mxcij}}` follows a Gumbel minimum distribution of mean :math:`0` and (estimated) parameter :math:`\frac{1}{\lambda}`.

Then, commuters choose the mode that **minimizes their transport cost** (according to the properties of the Gumbel distribution):

.. math::

    min_m [t_{mj}(x,c,w_{ic})] = -\frac{1}{\lambda}log(\sum_{m=1}^{M}exp[-\lambda\chi_i(\tau_m(x,c) + \delta_m(x,c)w_{ic})]) + \textcolor{orange}{\eta_{xcij}}

* :math:`\textcolor{orange}{\eta_{xcij}}` also follows a Gumbel minimum distribution of mean :math:`0` and scale parameter :math:`\frac{1}{\lambda}`.

In the ``import_transport_data`` function (``inputs.data`` module), for a given income group, this is imported with:

.. literalinclude:: ../inputs/data.py
   :language: python
   :lines: 2022-2025
   :lineno-start: 2022

Looking into the body of the ``compute_ODflows`` function (``calibration.sub.compute_income`` module), we see that the definitions of the ``transportCostModes`` and ``transportCost`` variables correspond to the above definitions of :math:`t_{mj}(x,c,w_{ic})` and :math:`min_m [t_{mj}(x,c,w_{ic})]`

.. literalinclude:: ../calibration/sub/compute_income.py
   :language: python
   :lines: 615-623
   :lineno-start: 615

.. literalinclude:: ../calibration/sub/compute_income.py
   :language: python
   :lines: 647-651
   :lineno-start: 647

Given their residential location :math:`x`, workers choose the workplace location :math:`c` that **maximizes their income net of commuting costs**:

.. math::

	max_c [y_{ic} - min_m [t_{mj}(x,c,w_{ic})]]

* :math:`y_{ic} = \chi_i w_{ic}` (``income_centers_init`` parameter)

This yields the **probability to choose to work** in location :math:`c` given residential location :math:`x`and income group :math:`i` (logit discrete choice model):

.. math::

    \pi_{c|ix} = \frac{exp[\lambda y_{ic} + log(\sum_{m=1}^{M}exp[-\lambda\chi_i(\tau_m(x,c) + \delta_m(x,c)w_{ic})])]}{\sum_{k=1}^{C} exp[\lambda y_{ik} + log(\sum_{m=1}^{M}exp[-\lambda\chi_i(\tau_m(x,k) + \delta_m(x,k)w_{ik})])]}

In the ``import_transport_data`` function, this corresponds to:

.. literalinclude:: ../inputs/data.py
   :language: python
   :lines: 2064-2072
   :lineno-start: 2064

From there, we calculate the **expected income net of commuting costs** for residents of group :math:`i` living in location :math:`x`:

.. math::

    \tilde{y}_i(x) = E[y_{ic} - min_m(t_m(x,c,w_{ic}))|x]

In the ``import_transport_data`` function, this corresponds to:

.. literalinclude:: ../inputs/data.py
   :language: python
   :lines: 2097-2099
   :lineno-start: 2097