.. _mixtures:

********
Mixtures
********

.. contents:: :depth: 2

Theoretical description
-----------------------
The mixture modeling used in CoolProp is based on the work of Kunz et al. :cite:`Kunz-BOOK-2007,Kunz-JCED-2012` and Lemmon :cite:`Lemmon-JPCRD-2000,Lemmon-JPCRD-2004,Lemmon-IJT-1999`

A mixture is composed of a number of components, and for each pair of components, it is necessary to have information for the excess Helmholtz energy term as well as the reducing function.  See below for what binary pairs are included in CoolProp.

The numerical methods required for mixtures are far more complicated than those for pure fluids, so the number of flash routines that are currently available are relatively small compared to pure fluids.

The only types of inputs that are allowed for mixtures right now are

- Pressure/quality
- Temperature/quality
- Temperature/pressure

.. Used in Python script later on
.. role:: raw-html(raw)
   :format: html

Binary pairs
------------

.. note::
   Please hover the mouse pointer over the coefficients to get the full accuracy
   for the listed coefficients. You can also get more information on references
   that are not in bibliography.

.. csv-table:: All binary pairs included in CoolProp
   :header-rows: 1
   :file: Mixtures.csv

Phase Envelope
--------------
.. plot::

    import CoolProp
    import matplotlib.pyplot as plt

    HEOS = CoolProp.AbstractState('HEOS','Methane&Ethane')
    for x0 in [0.02, 0.2, 0.4, 0.6, 0.8, 0.98]:
        HEOS.set_mole_fractions([x0, 1 - x0])
        try:
            HEOS.build_phase_envelope("dummy")
        except ValueError as VE:
            print(VE)
        PE = HEOS.get_phase_envelope_data()
        plt.plot(PE.T, PE.p, 'o-')

    plt.xlabel('Temperature [K]')
    plt.ylabel('Pressure [Pa]')
    plt.tight_layout()

Reducing Parameters
-------------------

From Lemmon :cite:`Lemmon-JPCRD-2000` for the properties of Dry Air, and also from Lemmon :cite:`Lemmon-JPCRD-2004` for the properties of R404A, R410A, etc.

.. math::

    \rho_r(\bar x) = \left[ \sum_{i=1}^m\frac{x_i}{\rho_{c_i}}+\sum_{i=1}^{m-1}\sum_{j=i+1}^{m}x_ix_j\zeta_{ij}\right]^{-1}

.. math::

    T_r(\bar x) = \sum_{i=1}^mx_iT_{c_i}+\sum_{i=1}^{m-1}\sum_{j=i+1}^mx_ix_j\xi_{ij}

From the GERG 2008 formulation :cite:`Kunz-JCED-2012`

.. math::

    T_r(\bar x) = \sum_{i=1}^{N}x_i^2T_{c,i} + \sum_{i=1}^{N-1}\sum_{j=i+1}^{N}2x_ix_j\beta_{T,ij}\gamma_{T,ij}\frac{x_i+x_j}{\beta_{T,ij}^2x_i+x_j}(T_{c,i}T_{c,j})^{0.5}
    
.. math::

    \frac{1}{\rho_r(\bar x)}=v_r(\bar x) = \sum_{i=1}^{N}x_i^2\frac{1}{\rho_{c,i}} + \sum_{i=1}^{N-1}\sum_{j=i+1}^N2x_ix_j\beta_{v,ij}\gamma_{v,ij}\frac{x_i+x_j}{\beta^2_{v,ij}x_i+x_j}\frac{1}{8}\left(\frac{1}{\rho_{c,i}^{1/3}}+\frac{1}{\rho_{c,j}^{1/3}}\right)^{3}
    
Excess Helmholtz Energy Terms
-----------------------------
From Lemmon :cite:`Lemmon-JPCRD-2004` for the properties of R404A, R410A, etc.

.. math::

    \alpha^E(\delta,\tau,\mathbf{x}) = \sum_{i=1}^{m-1} \sum_{j=i+1}^{m} \left [ x_ix_jF_{ij} \sum_{k}N_k\delta_{d_k}\tau^{t_k}\exp(-\delta^{l_k})\right]
    
where the terms :math:`N_k,d_k,t_k,l_k` correspond to the pair given by the indices :math:`i,j`

From Lemmon :cite:`Lemmon-JPCRD-2000` for the properties of Dry Air

.. math::

    \alpha^E(\delta,\tau,\mathbf{x}) = \left \lbrace \sum_{i=1}^{2} \sum_{j=i+1}^{3} x_ix_jF_{ij}\right\rbrace \left[-0.00195245\delta^2\tau^{-1.4}+0.00871334\delta^2\tau^{1.5} \right]


From Kunz and Wagner :cite:`Kunz-JCED-2012` for GERG 2008 formulation

.. math::

    \alpha^E(\delta,\tau,\mathbf{x}) = \sum_{i=1}^{N-1} \sum_{j=i+1}^{N} x_ix_jF_{ij}\alpha_{ij}^r(\delta,\tau)
    
where

.. math::

    \alpha_{ij}^r(\delta,\tau) = \sum_{k=1}^{K_{pol,ij}}\eta_{ij,k}\delta^{d_{ij,k}}\tau^{t_{ij,k}}+\sum_{k=K_{pol,ij}+1}^{K_{pol,ij}+K_{Exp,ij}}\eta_{ij,k}\delta^{d_{ij,k}}\tau^{t_{ij,k}}\exp[-\eta_{ij,k}(\delta-\varepsilon_{ij,k})^2-\beta_{ij,k}(\delta-\gamma_{ij,k})]
    
and is for the particular binary pair given by the indices :math:`i,j`.  This term is similar in form to other Helmholtz energy terms for pure fluids though the derivatives are slightly special.

Appendix
--------
To convert from the form from Lemmon for HFC and Air to that of GERG 2008, the following steps are required:

.. math::

    x_0T_{c0}+(1-x_0)T_{c1}+x_0(1-x_0)\xi_{01} = x_0^2T_{c0}+(1-x_0)^2T_{c1} + 2x_0(1-x_0)\beta\gamma_T\frac{x_0+(1-x_0)}{\beta x_0 + (1-x_0)}\sqrt{T_{c0}T_{c1}}
    
set :math:`\beta=1`, solve for :math:`\gamma`.  Equate the terms

.. math::

    x_0T_{c0}+(1-x_0)T_{c1}+x_0(1-x_0)\xi_{01} = x_0^2T_{c0}+(1-x_0)^2T_{c1} + 2x_0(1-x_0)\gamma_T\sqrt{T_{c0}T_{c1}}
    
Move to LHS

.. math::

    [x_0-x_0^2]T_{c0}+[(1-x_0)-(1-x_0)^2]T_{c1}+x_0(1-x_0)\xi_{01} = 2x_0(1-x_0)\gamma_T\sqrt{T_{c0}T_{c1}}

Factor

.. math::

    x_0(1-x_0)T_{c0}+(1-x_0)[1-(1-x_0)]T_{c1}+x_0(1-x_0)\xi_{01} = 2x_0(1-x_0)\gamma_T\sqrt{T_{c0}T_{c1}}
    
Expand

.. math::

    x_0(1-x_0)T_{c0}+x_0(1-x_0)T_{c1}+x_0(1-x_0)\xi_{01} = 2x_0(1-x_0)\gamma_T\sqrt{T_{c0}T_{c1}}
    
Cancel factors of :math:`x_0(1-x_0)`

.. math::

    T_{c0}+T_{c1}+\xi_{01} = 2\gamma_T\sqrt{T_{c0}T_{c1}}
    
Answer:

.. math::

    \boxed{\gamma_T = \dfrac{T_{c0}+T_{c1}+\xi_{01}}{2\sqrt{T_{c0}T_{c1}}}}
    
Same idea for the volume

.. math::

    \boxed{\gamma_v = \dfrac{v_{c0}+v_{c1}+\zeta_{01}}{\frac{1}{4}\left(\frac{1}{\rho_{c,i}^{1/3}}+\frac{1}{\rho_{c,j}^{1/3}}\right)^{3}}}

References
----------
:ref:`Go to the bibliography <bibliography>`