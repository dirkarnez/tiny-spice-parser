tiny-spice-parser
=================

### TODO
- [ ] to calculation base on the spice
  - ChatGPT `VAF + (log(IC/IS) - 1) * (BF / RB + VJE)`?
- [ ] Study microcap
  - https://www.iee.et.tu-dresden.de/~jmueller/simulation/soft/microcap/MC12.RefManual.pdf
- [ ] Study Kicad
  - **https://github.com/KiCad/kicad-source-mirror/tree/master/eeschema/sim**
- [ ] Constraint programming
- [ ] https://github.com/urish/circuit-sandbox!!!!!!!!!!!!!!!!!!!!!!
```js
	///////////////////////////////////////////////////////////////////////////////
	//
	//  Very basic Ebers-Moll BJT model
	//
	///////////////////////////////////////////////////////////////////////////////

    function bjt(c,b,e,area,Ics,Ies,af,ar,name,type) {
	    Device.call(this);
	    this.e = e;
	    this.b = b;
	    this.c = c;
	    this.name = name;
	    this.af = af;
	    this.ar = ar;
	    this.area = area;
	    this.aIcs = this.area*Ics;
        this.aIes = this.area*Ies;
	    if (type != 'n' && type != 'p') { 
	    	throw 'BJT type is not npn or pnp';
	    }
	    this.type_sign = (type == 'n') ? 1 : -1;
	    this.vt = 0.026;
	    this.leakCond = 1.0e-12;
	}
	bjt.prototype = new Device();
        bjt.prototype.constructor = bjt;

        bjt.prototype.load_linear = function(ckt) {
	    // bjt's are nonlinear, just like javascript progammers
	};

        bjt.prototype.load_dc = function(ckt,soln,rhs) {
	    let e = this.e; let b = this.b; let c = this.c;
	    let vbc = this.type_sign * ckt.get_two_terminal(b, c, soln);
	    let vbe = this.type_sign * ckt.get_two_terminal(b, e, soln);
        let IrGr = diodeEval(vbc, this.vt, this.aIcs);
        let IfGf = diodeEval(vbe, this.vt, this.aIes);

        // Sign convention is emitter and collector currents are leaving.
        let ie = this.type_sign * (IfGf[0] - this.ar*IrGr[0]);
        let ic = this.type_sign * (IrGr[0] - this.af*IfGf[0]);
        let ib = -(ie+ic);  		//current flowing out of base

	    ckt.add_to_rhs(b,ib,rhs);  	//current flowing out of base
	    ckt.add_to_rhs(c,ic,rhs);  	//current flowing out of collector
	    ckt.add_to_rhs(e,ie,rhs);   //and out emitter
	    ckt.add_conductance(b,e,IfGf[1]);
	    ckt.add_conductance(b,c,IrGr[1]);
	    ckt.add_conductance(c,e,this.leakCond);

	    ckt.add_to_G(b, c, this.ar*IrGr[1]);
	    ckt.add_to_G(b, e, this.af*IfGf[1]);	    
	    ckt.add_to_G(b, b, -(this.af*IfGf[1] + this.ar*IrGr[1]));
	    
	    ckt.add_to_G(e, b, this.ar*IrGr[1]);
	    ckt.add_to_G(e, c, -this.ar*IrGr[1]);
	    
	    ckt.add_to_G(c, b, this.af*IfGf[1]);
	    ckt.add_to_G(c, e, -this.af*IfGf[1]);
	};

        bjt.prototype.load_tran = function(ckt,soln,crnt,chg,time) {
	    this.load_dc(ckt,soln,crnt,crnt);
	};

	bjt.prototype.load_ac = function(ckt) {
	};
```

### (P)SPICE grammar
- https://www.orcad.com/pspice-free-trial
- https://www.seas.upenn.edu/~jan/spice/PSpice_ReferenceguideOrCAD.pdf
- https://engineering.purdue.edu/~ee255d3/files/PSPICEtutorial.pdf
- https://www.ti.com/lit/an/sloa070/sloa070.pdf
- [Basic SPICE Simulation Model Parameters - NI](https://www.ni.com/en/shop/electronic-test-instrumentation/application-software-for-electronic-test-and-instrumentation-category/what-is-multisim/spice-simulation-fundamentals/basic-spice-simulation-model-parameters-.html)
- https://www.ece.mcgill.ca/~grober4/SPICE/SPICE_Decks/1st_Edition_LTSPICE/chapter4/Chapter%204%20BJTs%20web%20version.html
- https://www.mikrocontroller.net/attachment/168555/Modeling_BJTs_in_Multisim.pdf
- https://resources.pcb.cadence.com/blog/2020-determining-spice-model-parameters-for-transistors-easily-and-accurately
- [BJT](https://help.altair.com/activate/help/en_us/block_reference_guide/_mo/_lib/Spice/HTML/bjt_t.html)
- [docs/spice_parameters.pdf](docs/spice_parameters.pdf)
- https://web.stanford.edu/class/ee133/handouts/general/spice_ref.pdf
- https://ltwiki.org/files/SPICEdiodeModel.pdf
- https://www.acsu.buffalo.edu/~wie/applet/spice_pndiode/spice_diode_table.html
- [Diode Spice Model Parameters Explained - Free Online PCB CAD Library](https://www.ultralibrarian.com/2024/04/04/diode-spice-model-parameters-explained-ulc)
- [Circuit sandbox | Spinning Numbers](https://spinningnumbers.org/a/circuit-sandbox.html#diode-model)
- [Analysis of Circuits - Intro](https://lpsa.swarthmore.edu/Systems/Electrical/mna/MNA1.html)

### BJT model
- https://techdocs.altium.com/display/AMSE/Bipolar+Junction+Transistor+(BJT)+Model
- **https://www.mikrocontroller.net/attachment/168555/Modeling_BJTs_in_Multisim.pdf**


### onsemi
- https://www.onsemi.com/design/resources/technical-documentation
  - select "Data Sheets" and "Simulation Models" before search
  - spice and datasheet may not be aligned
### AD
- https://www.analog.com/en/products/ad8275.html
  - both spice and datasheet

### Netlist
- https://github.com/dan-fritchman/Netlist

### Model files / circuit
- https://github.com/killerzman/OrCAD-AudioPowerAmplifier
- https://github.com/NikolaosGian/opAmp
- https://github.com/ujjawalece/Biomedical-Data-Acquisition-ECG
- https://github.com/Vladi2017/OrCAD_PSpice_1

### Extra
- monte carlo
  - https://resources.pcb.cadence.com/blog/2019-monte-carlo-analysis-and-simulation-for-electronic-circuits

### Reference
- https://github.com/SpiceSharp/SpiceSharpParser
- http://www.wrcad.com/manual/wrsmanual/node1.html
- https://www.pspice.com/amplifiers-and-linear-ics/operational-amplifiers/cmos
- https://github.com/ottoragam/Orcad-libraries
- https://github.com/Werni2A/OpenOrCadParser/tree/main
