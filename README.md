# EPITRABAJO2025

/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */

package com.mycompany.epitrabajo2025;
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */


import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.*;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;

/**
 * Clase POO: Implementa Parámetros de Simulación que ahora se CALCULARÁN.
 */
class ParametrosSimulados {
    private final double temperatura; // T (°C)
    private final double presion; // P (atm)
    private final double areaElectrodo; // A (cm²)

    // Constructor
    public ParametrosSimulados(double temp, double pres, double area) {
        this.temperatura = temp;
        this.presion = pres;
        this.areaElectrodo = area;
    }

    // Getters
    public double getTemperatura() { return temperatura; }
    public double getPresion() { return presion; }
    public double getAreaElectrodo() { return areaElectrodo; }

    /**
     * Calcula la densidad de corriente (I/A).
     * @param corriente Corriente aplicada (A).
     * @return Densidad de corriente (A/cm²).
     */
    public double calcularDensidadCorriente(double corriente) {
        if (areaElectrodo <= 0) return 0.0;
        return corriente / areaElectrodo;
    }
}


public class EPITRABAJO2025 extends JFrame { // CLASE PRINCIPAL

    // Inputs restantes
    private final JTextField txtCorriente, txtVoltaje, txtTiempo;
    private final JComboBox<String> comboUnidadTiempo;
    private final JTextArea resultadosArea;

    // Unit mode: 0 = H2(g)/CO2(kg), 1 = all g, 2 = all kg
    private int modoUnidad = 0;

    // Model + panels
    private ModeloElectrolisis modelo;
    private final MiPanelGraficoBarras panelBarras;
    private final MiPanelGraficoTorta panelTorta;
    private final MiPanelGraficoEvolucion panelEvolucion;
    private final MiPanelEcuacionQuimica panelEcuacion;

    private final DecimalFormat df2 = new DecimalFormat("#,##0.00");
    private final DecimalFormat df3 = new DecimalFormat("#,##0.000");

    public EPITRABAJO2025() {
        setTitle("🔬 EPITRABAJO2025 - Electrólisis del Agua (Simulador Derivado)");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(1500, 900);
        setLocationRelativeTo(null);
        setLayout(new BorderLayout(10, 10));
        getContentPane().setBackground(new Color(225, 240, 250));

        // Header
        JLabel lblTitulo = new JLabel(" EPI H₂ Visual Lab — Simulación de Parámetros Derivados", JLabel.CENTER);
        lblTitulo.setFont(new Font("Segoe UI", Font.BOLD, 26));
        lblTitulo.setForeground(new Color(0, 70, 170));
        lblTitulo.setBorder(BorderFactory.createEmptyBorder(8, 0, 8, 0));
        add(lblTitulo, BorderLayout.NORTH);

        // Left inputs - SOLO CORRIENTE, VOLTAJE, TIEMPO
        JPanel panelParametros = new JPanel(new GridBagLayout());
        panelParametros.setBackground(new Color(245, 250, 255));
        panelParametros.setBorder(BorderFactory.createTitledBorder(
                BorderFactory.createLineBorder(new Color(0, 110, 215), 2, true),
                " Entrada de Datos (Básicos)",
                0, 0, new Font("Segoe UI", Font.BOLD, 14), new Color(0, 90, 180)
        ));

        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(8, 10, 8, 10);
        gbc.anchor = GridBagConstraints.WEST;
        gbc.fill = GridBagConstraints.HORIZONTAL;

        // --- INICIALIZACIÓN DE CAMPOS RESTANTES ---
        txtCorriente = new JTextField("", 10);
        txtVoltaje = new JTextField("", 10);
        txtTiempo = new JTextField("", 10);
        
        comboUnidadTiempo = new JComboBox<>(new String[]{"Segundos", "Minutos", "Horas"});

        int row = 0;
        gbc.gridx = 0; gbc.gridy = row; panelParametros.add(new JLabel("Corriente (A):"), gbc);
        gbc.gridx = 1; panelParametros.add(txtCorriente, gbc);
        row++; gbc.gridx = 0; gbc.gridy = row; panelParametros.add(new JLabel("Voltaje (V):"), gbc);
        gbc.gridx = 1; panelParametros.add(txtVoltaje, gbc);
        row++; gbc.gridx = 0; gbc.gridy = row; panelParametros.add(new JLabel("Tiempo:"), gbc);
        gbc.gridx = 1; panelParametros.add(txtTiempo, gbc);
        row++; gbc.gridx = 0; gbc.gridy = row; panelParametros.add(new JLabel("Unidad tiempo:"), gbc);
        gbc.gridx = 1; panelParametros.add(comboUnidadTiempo, gbc);
        
        // Espacio para mantener el layout (relleno donde estaban los otros inputs)
        for(int i = 0; i < 4; i++){
            row++; gbc.gridx = 0; gbc.gridy = row; panelParametros.add(new JLabel(""), gbc);
        }
        
        JPanel leftWrapper = new JPanel(new BorderLayout());
        leftWrapper.setBackground(new Color(225, 240, 250));
        leftWrapper.add(panelParametros, BorderLayout.NORTH);
        add(leftWrapper, BorderLayout.WEST);

        // Results on right
        resultadosArea = new JTextArea(28, 44);
        resultadosArea.setEditable(false);
        resultadosArea.setFont(new Font("Consolas", Font.PLAIN, 13));
        resultadosArea.setLineWrap(true);
        resultadosArea.setWrapStyleWord(true);
        resultadosArea.setMargin(new Insets(12, 12, 12, 12));
        JScrollPane spResults = new JScrollPane(resultadosArea);
        spResults.setBorder(BorderFactory.createTitledBorder(
                BorderFactory.createLineBorder(new Color(0, 150, 90), 2, true),
                " Resultados",
                0, 0, new Font("Segoe UI", Font.BOLD, 14), new Color(0,120,70)
        ));
        add(spResults, BorderLayout.EAST);

        // Center: tabs + unit buttons ABOVE graphs (option B)
        JTabbedPane tabbed = new JTabbedPane();
        panelBarras = new MiPanelGraficoBarras();
        panelTorta = new MiPanelGraficoTorta();
        panelEvolucion = new MiPanelGraficoEvolucion();
        panelEcuacion = new MiPanelEcuacionQuimica();

        tabbed.addTab("Barras", panelBarras);
        tabbed.addTab("Torta", panelTorta);
        tabbed.addTab("Evolución", panelEvolucion);
        tabbed.addTab("Ecuación", panelEcuacion);
        tabbed.setFont(new Font("Segoe UI", Font.PLAIN, 13));

        // Unit buttons above tabs
        JPanel panelUnitTop = new JPanel(new FlowLayout(FlowLayout.LEFT));
        panelUnitTop.setBackground(new Color(225, 240, 250));
        JLabel lblUnit = new JLabel("Mostrar unidades:");
        lblUnit.setFont(new Font("Segoe UI", Font.PLAIN, 13));
        JButton b0 = crearBotonModo("H₂ g / CO₂ kg", 0);
        JButton b1 = crearBotonModo("Todo en g", 1);
        JButton b2 = crearBotonModo("Todo en kg", 2);
        panelUnitTop.add(lblUnit);
        panelUnitTop.add(b0); panelUnitTop.add(b1); panelUnitTop.add(b2);

        JPanel centerPanel = new JPanel(new BorderLayout());
        centerPanel.add(panelUnitTop, BorderLayout.NORTH);
        centerPanel.add(tabbed, BorderLayout.CENTER);
        add(centerPanel, BorderLayout.CENTER);

        // Bottom: sim / reset buttons
        JPanel bottom = new JPanel();
        bottom.setBackground(new Color(225, 240, 250));
        JButton btnSim = crearBotonRedondeado(" Ejecutar Simulación", new Color(0,170,80));
        JButton btnReset = crearBotonRedondeado(" Reiniciar", new Color(220,70,70));
        btnSim.addActionListener(new SimulacionListener());
        btnReset.addActionListener(e -> resetearCampos());
        bottom.add(btnSim);
        bottom.add(Box.createHorizontalStrut(12));
        bottom.add(btnReset);
        add(bottom, BorderLayout.SOUTH);

        setVisible(true);
    }

    // Helper: create unit mode button
    private JButton crearBotonModo(String texto, int modo) {
        JButton b = new JButton(texto);
        b.setFont(new Font("Segoe UI", Font.PLAIN, 12));
        b.setBackground(new Color(200,220,255));
        b.setFocusPainted(false);
        b.addActionListener(e -> {
            modoUnidad = modo;
            actualizarResultados(); // refresh displayed units and repaint
        });
        return b;
    }

    // Helper: rounded button
    private JButton crearBotonRedondeado(String texto, Color color) {
        JButton b = new JButton(texto) {
            protected void paintComponent(Graphics g) {
                Graphics2D g2 = (Graphics2D) g.create();
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
                g2.setColor(color);
                g2.fillRoundRect(0, 0, getWidth(), getHeight(), 18, 18);
                super.paintComponent(g);
                g2.dispose();
            }
        };
        b.setFont(new Font("Segoe UI", Font.BOLD, 14));
        b.setForeground(Color.WHITE);
        b.setFocusPainted(false);
        b.setContentAreaFilled(false);
        b.setOpaque(false);
        b.setPreferredSize(new Dimension(200, 38));
        return b;
    }

    // Simulation action
    private class SimulacionListener implements ActionListener {
        public void actionPerformed(ActionEvent e) {
            try {
                // Validación para evitar NumberFormatException con campos vacíos
                if (txtCorriente.getText().trim().isEmpty() || txtVoltaje.getText().trim().isEmpty() ||
                    txtTiempo.getText().trim().isEmpty()) {
                    throw new NumberFormatException(); // Lanzar excepción si falta algún valor
                }
                
                double I = Double.parseDouble(txtCorriente.getText().trim());
                double V = Double.parseDouble(txtVoltaje.getText().trim());
                double t = Double.parseDouble(txtTiempo.getText().trim());
                
                String unidad = (String) comboUnidadTiempo.getSelectedItem();
                if ("Minutos".equals(unidad)) t *= 60;
                if ("Horas".equals(unidad)) t *= 3600;

                // Ahora ModeloElectrolisis calcula los parámetros derivados
                modelo = new ModeloElectrolisis(I, V, t); 
                modelo.calcularResultados();
                actualizarResultados();
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(EPITRABAJO2025.this,
                        "Por favor, ingrese valores numéricos válidos en todos los campos de entrada (Corriente, Voltaje, Tiempo).",
                        "Error de Entrada", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    // Update results + repaint charts
    private void actualizarResultados() {
        if (modelo == null) return;

        double h2 = modelo.getHidrogenoProducido(); // grams
        double co2 = modelo.getEmisionesEvitadas(); // kg
        String uH2 = "g", uCO2 = "kg";

        if (modoUnidad == 1) { // all grams
            co2 *= 1000.0; uCO2 = "g";
        } else if (modoUnidad == 2) { // all kg
            h2 /= 1000.0; uH2 = "kg";
        }

        ParametrosSimulados params = modelo.getParametros();
        double densidad = params.calcularDensidadCorriente(modelo.I);
        double ef_calc = modelo.getEficienciaCalculada() * 100.0;


        StringBuilder sb = new StringBuilder();
        sb.append("=============================================\n");
        sb.append("RESULTADOS DE LA SIMULACIÓN\n");
        sb.append("=============================================\n\n");
        sb.append("Ecuación: 2H₂O(l) → 2H₂(g) + O₂(g)\n\n");
        sb.append("Corriente: ").append(df2.format(modelo.I)).append(" A\n");
        sb.append("Voltaje: ").append(df2.format(modelo.V)).append(" V\n");
        sb.append("Tiempo: ").append(df2.format(modelo.t / 3600.0)).append(" h\n\n");
        
        sb.append("--- Parámetros Derivados (Calculados) ---\n");
        sb.append("Eficiencia (Ajustada): ").append(df2.format(ef_calc)).append(" %\n");
        sb.append("Temperatura (Derivada): ").append(df2.format(params.getTemperatura())).append(" °C\n");
        sb.append("Presión (Derivada): ").append(df2.format(params.getPresion())).append(" atm\n"); 
        sb.append("Área Electrodo (Derivada): ").append(df2.format(params.getAreaElectrodo())).append(" cm²\n");
        sb.append("Densidad de Corriente (I/A): ").append(df3.format(densidad)).append(" A/cm²\n\n");
        sb.append("-------------------------------------------\n\n");
        
        sb.append("Hidrógeno producido: ").append(df3.format(h2)).append(" ").append(uH2).append("\n");
        sb.append("Energía consumida: ").append(df3.format(modelo.getEnergiaConsumida())).append(" kWh\n");
        sb.append("CO₂ evitado: ").append(df3.format(co2)).append(" ").append(uCO2).append("\n\n");
        sb.append("Eficiencia Energética Total: ").append(df2.format(modelo.getEficiencia())).append(" %\n");

        resultadosArea.setText(sb.toString());

        panelBarras.repaint();
        panelTorta.repaint();
        panelEvolucion.setSeries(modelo.generateH2Timeline(modoUnidad));
        panelEvolucion.repaint();
    }

    private void resetearCampos() {
        // Reinicio a valores VACÍOS
        txtCorriente.setText("");
        txtVoltaje.setText("");
        txtTiempo.setText("");
        
        resultadosArea.setText("");
        modelo = null;
        panelBarras.repaint();
        panelTorta.repaint();
        panelEvolucion.clearSeries();
        panelEvolucion.repaint();
    }

    /* ===========================
      ModeloElectrolisis (interno) - LÓGICA DE CÁLCULO MODIFICADA
      =========================== */
    private class ModeloElectrolisis {
        final double FARADAY = 96485.3329;
        final double M_H2 = 2.016; // g/mol
        final double EM_CO2 = 0.233; // kg CO2 / kWh avoided
        final double E_H2_KWH_G = 0.033; // kWh per gram H2 (theoretical)
        
        // Parámetros derivados
        private double ef_calc; // Eficiencia ajustada (fracción)
        
        // CORRECCIÓN: Se elimina 'final'
        private ParametrosSimulados parametros; 

        double I, V, t; // A, V, seconds
        double h2; // grams
        double eCons; // kWh
        double co2; // kg
        double eTheo; // kWh
        double eta; // % (Eficiencia energética total)

        ModeloElectrolisis(double I, double V, double t) {
            this.I = I; this.V = V; this.t = t;
            this.parametros = new ParametrosSimulados(0, 0, 0); 
        }
        
        public ParametrosSimulados getParametros() { return parametros; }
        public double getEficienciaCalculada() { return ef_calc; }


        /**
         * Lógica de derivación: Calcula la eficiencia y los parámetros 
         * simulados (T, P, A) a partir de I, V, t.
         */
        void calcularResultados() {
            // --- 1. Cálculo de Parámetros Derivados a partir de I, V, t ---
            
            // A. Cálculo de la Eficiencia Ajustada (ef_calc)
            double V_Ref = 1.45; // Voltaje de operación de referencia típico
            double Ef_Base = 0.85; // Eficiencia base

            if (V <= 1.23) { 
                ef_calc = 0.0;
            } else {
                double relV = Math.min(1.0, V_Ref / V);
                ef_calc = Ef_Base * relV;
            }
            
            // B. Cálculo de Temperatura, Presión y Área (Dependientes de I, t y V)
            
            double potencia = V * I; // Watts (J/s)
            double temp_calc = 25.0 + (potencia / 100.0); 
            temp_calc = Math.min(temp_calc, 95.0); 

            double carga = I * t;
            double presion_calc = 1.0 + (carga / 100000.0);
            presion_calc = Math.min(presion_calc, 15.0); 

            double dens_ref = 0.2; 
            double area_calc = I / dens_ref; 
            
            ParametrosSimulados params_calc = new ParametrosSimulados(temp_calc, presion_calc, area_calc);
            
            // --- 2. Recalcular la Eficiencia Ajustada Final (Por si la temperatura afecta) ---
            double factorTemp = 1.0 + 0.003 * (params_calc.getTemperatura() - 25.0);
            double efAdj = ef_calc * factorTemp;
            if (efAdj > 1.0) efAdj = 1.0;
            this.ef_calc = efAdj;

            // --- 3. Cálculos de Producción y Energía ---
            
            double molesH2 = (carga / (2.0 * FARADAY)) * efAdj; 
            h2 = molesH2 * M_H2; 

            double energiaJ = V * I * t;
            eCons = energiaJ / 3600000.0; 
            co2 = eCons * EM_CO2; 
            eTheo = h2 * E_H2_KWH_G;
            eta = (eCons > 0) ? Math.min(100.0, (eTheo / eCons) * 100.0) : 0.0;

            // ASIGNACIÓN CORREGIDA
            this.parametros = params_calc; 
        }

        List<Point2D.Double> generateH2Timeline(int modoUnidadLocal) {
            List<Point2D.Double> series = new ArrayList<>();
            if (t <= 0) { series.add(new Point2D.Double(0, 0)); return series; }

            double samples = 120.0; 
            double dt = Math.max(1.0, t / samples);
            double molesPerSec = (I / (2.0 * FARADAY)) * this.ef_calc; 
            double gPerSec = molesPerSec * M_H2;
            double cumulative = 0.0;
            
            for (double time = 0.0; time < t; time += dt) {
                cumulative += gPerSec * dt;
                double display = cumulative;
                if (modoUnidadLocal == 2) display = cumulative / 1000.0; 
                series.add(new Point2D.Double(time, display));
            }

            double finalVal = h2;
            if (modoUnidadLocal == 2) finalVal /= 1000.0;
            series.add(new Point2D.Double(t, finalVal));
            return series;
        }

        // getters
        double getHidrogenoProducido() { return h2; } // g
        double getEnergiaConsumida() { return eCons; } // kWh
        double getEmisionesEvitadas() { return co2; } // kg
        double getEnergiaTeorica() { return eTheo; } // kWh
        double getEficiencia() { return eta; } // %
    }

    /* ===========================
      Panels gráficos (sin cambios relevantes)
      =========================== */
    
    private class MiPanelGraficoBarras extends JPanel {
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);
            setBackground(new Color(245, 250, 255));
            Graphics2D g2 = (Graphics2D) g;
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

            int w = getWidth();
            int h = getHeight();

            int margin = 60; 
            int axisOffset = 30; 

            int plotLeft = margin; 
            int plotRight = w - margin;
            int plotTop = margin + axisOffset; 
            int plotBottom = h - margin - axisOffset; 
            int plotW = plotRight - plotLeft;
            int plotH = plotBottom - plotTop;

            g2.setColor(new Color(230,230,230));
            for (int x = plotLeft + axisOffset; x <= plotRight; x += 40) g2.drawLine(x, plotTop, x, plotBottom);
            for (int y = plotBottom; y >= plotTop; y -= 40) g2.drawLine(plotLeft + axisOffset, y, plotRight, y);

            g2.setColor(Color.DARK_GRAY);
            g2.setStroke(new BasicStroke(2f));
            g2.drawLine(plotLeft + axisOffset, plotBottom, plotRight, plotBottom); 
            g2.drawLine(plotLeft + axisOffset, plotBottom, plotLeft + axisOffset, plotTop); 

            g2.setFont(new Font("Segoe UI", Font.BOLD, 14));
            g2.drawString("Producción de H₂ y CO₂ evitado", plotLeft + axisOffset, margin - 15); 

            g2.setFont(new Font("Segoe UI", Font.PLAIN, 12));
            g2.setColor(Color.BLACK);
            g2.drawString("Producto", plotRight - 60, plotBottom + 40);
            String yLabel = (modoUnidad == 2) ? "Masa (kg)" : "Masa (g)";
            g2.drawString(yLabel, plotLeft, plotTop - 10);

            if (modelo == null) return;

            double h2Val = modelo.getHidrogenoProducido(); 
            double co2Val = modelo.getEmisionesEvitadas(); 
            String unitH2 = "g", unitCO2 = "kg";

            if (modoUnidad == 1) { co2Val *= 1000.0; unitCO2 = "g"; }
            if (modoUnidad == 2) { h2Val /= 1000.0; unitH2 = "kg"; }

            double maxVal = Math.max(h2Val, co2Val);
            if (maxVal <= 0) maxVal = 1.0;

            int barWidth = Math.min(80, plotW/8);
            int xCenter = plotLeft + axisOffset + plotW/2;
            int xH2 = xCenter - (plotW/4) - barWidth/2;
            int xCO2 = xCenter + (plotW/4) - barWidth/2;
            int hH2 = (int) ((h2Val / maxVal) * plotH);
            int hCO2 = (int) ((co2Val / maxVal) * plotH);

            g2.setColor(new Color(0,120,255));
            g2.fillRoundRect(xH2, plotBottom - hH2, barWidth, hH2, 10, 10);
            g2.setColor(new Color(0,180,80));
            g2.fillRoundRect(xCO2, plotBottom - hCO2, barWidth, hCO2, 10, 10);

            g2.setColor(Color.BLACK);
            g2.setFont(new Font("Segoe UI", Font.BOLD, 12));
            g2.drawString(df3.format(h2Val) + " " + unitH2, xH2, plotBottom - hH2 - 8);
            g2.drawString(df3.format(co2Val) + " " + unitCO2, xCO2, plotBottom - hCO2 - 8);

            int lx = plotRight - 180, ly = margin - 15;
            g2.setColor(new Color(0,120,255)); g2.fillOval(lx, ly, 10, 10); g2.setColor(Color.BLACK);
            g2.drawString("H₂ producido", lx + 14, ly + 9);
            g2.setColor(new Color(0,180,80)); g2.fillOval(lx, ly + 18, 10, 10); g2.setColor(Color.BLACK);
            g2.drawString("CO₂ evitado", lx + 14, ly + 27);
        }
    }

    private class MiPanelGraficoTorta extends JPanel {
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);
            setBackground(new Color(255,255,255));
            if (modelo == null) return;
            Graphics2D g2 = (Graphics2D) g;
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

            int w = getWidth(), h = getHeight();
            int size = Math.min(w, h) / 3;
            int x = 40;
            int yStart = 60; 

            double eUtil = modelo.getEnergiaTeorica();
            double eTot = modelo.getEnergiaConsumida();
            double lost = Math.max(0.0, eTot - eUtil);
            if (eTot <= 0) return;
            int angUtil = (int) Math.round((eUtil / eTot) * 360.0);

            g2.setColor(new Color(0,180,80)); g2.fillArc(x, yStart, size, size, 0, angUtil);
            g2.setColor(new Color(220,60,60)); g2.fillArc(x, yStart, size, size, angUtil, 360 - angUtil);

            g2.setColor(Color.BLACK); g2.setFont(new Font("Segoe UI", Font.BOLD, 14));
            g2.drawString("Comparativa Energética", x, yStart - 20); 
            g2.setFont(new Font("Segoe UI", Font.PLAIN, 12));
            int lx = x + size + 20, ly = yStart + 10;
            g2.setColor(new Color(0,180,80)); g2.fillOval(lx, ly, 10, 10); g2.setColor(Color.BLACK);
            g2.drawString("Energía útil: " + df3.format(eUtil) + " kWh", lx + 14, ly + 9);
            g2.setColor(new Color(220,60,60)); g2.fillOval(lx, ly + 20, 10, 10); g2.setColor(Color.BLACK);
            g2.drawString("Pérdidas: " + df3.format(lost) + " kWh", lx + 14, ly + 29);

            g2.drawString("Útil: " + df2.format((eUtil/eTot)*100) + "%", lx, ly + 60);
            g2.drawString("Pérdida: " + df2.format((lost/eTot)*100) + "%", lx + 120, ly + 60);
        }
    }

    private class MiPanelGraficoEvolucion extends JPanel {
        private List<Point2D.Double> series = new ArrayList<>();
        private String unit = "g";

        void setSeries(List<Point2D.Double> s) { series = s; }
        void clearSeries() { series = new ArrayList<>(); }

        protected void paintComponent(Graphics g) {
            super.paintComponent(g);
            setBackground(new Color(245,250,255));
            Graphics2D g2 = (Graphics2D) g;
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

            int w = getWidth(), h = getHeight();
            
            int margin = 70; 
            int axisOffset = 20; 
            
            int left = margin; 
            int right = w - margin; 
            int top = margin + axisOffset; 
            int bottom = h - margin - 40; 
            int plotW = right - left - axisOffset; 
            int plotH = bottom - top;

            g2.setColor(new Color(230,230,230));
            for (int x = left + axisOffset; x <= right; x += 50) g2.drawLine(x, top, x, bottom);
            for (int y = bottom; y >= top; y -= 40) g2.drawLine(left + axisOffset, y, right, y);

            g2.setColor(Color.DARK_GRAY); g2.setStroke(new BasicStroke(2f));
            g2.drawLine(left + axisOffset, bottom, right, bottom); 
            g2.drawLine(left + axisOffset, bottom, left + axisOffset, top); 

            g2.setFont(new Font("Segoe UI", Font.BOLD, 14));
            g2.drawString("Evolución: Hidrógeno producido vs Tiempo", left + axisOffset, margin - 15); 

            g2.setFont(new Font("Segoe UI", Font.PLAIN, 12)); g2.setColor(Color.BLACK);
            g2.drawString("Tiempo (s)", right - 60, bottom + 30);
            String ylabel = (modoUnidad == 2) ? "H₂ (kg)" : "H₂ (g)";
            g2.drawString(ylabel, left, top - 6);

            if (series == null || series.isEmpty()) return;

            double maxT = modelo.t; 
            double maxM = 0;
            for (Point2D.Double p : series) if (p.y > maxM) maxM = p.y;
            if (maxM <= 0) maxM = 1.0;

            g2.setColor(new Color(0,120,255)); g2.setStroke(new BasicStroke(2f));
            int prevX = -1, prevY = -1;
            for (Point2D.Double p : series) {
                int px = left + axisOffset + (int)((p.x / maxT) * plotW);
                int py = bottom - (int)((p.y / maxM) * plotH);
                if (prevX != -1) g2.drawLine(prevX, prevY, px, py);
                prevX = px; prevY = py;
                g2.fill(new Ellipse2D.Double(px-2, py-2, 4, 4));
            }

            Point2D.Double last = series.get(series.size()-1);
            g2.setFont(new Font("Segoe UI", Font.PLAIN, 12));
            g2.drawString("Total: " + df3.format(last.y) + ((modoUnidad==2)?" kg":" g"), right - 180, top + 14);
        }
    }

    private class MiPanelEcuacionQuimica extends JPanel {
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);
            setBackground(new Color(245,250,255));
            g.setFont(new Font("Segoe UI", Font.BOLD, 18));
            g.setColor(new Color(0,90,190));
            g.drawString("Ecuación: 2H₂O(l) → 2H₂(g) + O₂(g)", 40, 60);
            g.setFont(new Font("Segoe UI", Font.PLAIN, 13));
            g.setColor(Color.DARK_GRAY);
            g.drawString("La electrólisis separa el agua en hidrógeno y oxígeno usando electricidad.", 40, 100);
            g.drawString("Los parámetros (Eficiencia, T, P, A) son *derivados* de V, I, t en este modelo.", 40, 124);
            g.drawString("Se asumen relaciones funcionales entre V, I, t y los parámetros simulados.", 40, 148);
        }
    }

    // Main
    public static void main(String[] args) {
        SwingUtilities.invokeLater(EPITRABAJO2025::new);
    }
}
