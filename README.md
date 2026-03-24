<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>OncoSim | Chemotherapy Preparation Simulator</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.11.4/gsap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.11.4/ScrollTrigger.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
            -webkit-touch-callout: none;
            user-select: none;
            touch-action: manipulation;
        }

        :root {
            --primary: #1a73e8;
            --secondary: #0d47a1;
            --success: #00c853;
            --warning: #ffab00;
            --danger: #ff1744;
            --dark: #121212;
            --darker: #0a0a0a;
            --light: #f8f9fa;
            --gray: #5f6368;
            --sterile-blue: #e3f2fd;
            --contamination-red: #ffebee;
            --shadow-sm: 0 2px 8px rgba(0,0,0,0.15);
            --shadow-md: 0 4px 20px rgba(0,0,0,0.25);
            --shadow-lg: 0 8px 40px rgba(0,0,0,0.35);
            --transition: all 0.35s cubic-bezier(0.165, 0.84, 0.44, 1);
            --border-radius: 16px;
            --border-radius-lg: 24px;
            --border-radius-xl: 32px;
            --glass-bg: rgba(255, 255, 255, 0.08);
            --glass-border: rgba(255, 255, 255, 0.12);
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, var(--darker), #0f1c33);
            color: var(--light);
            overflow: hidden;
            height: 100vh;
            width: 100vw;
            position: relative;
            touch-action: none;
        }

        body::before {
            content: "";
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: 
                radial-gradient(circle at 10% 20%, rgba(26, 115, 232, 0.15) 0%, transparent 25%),
                radial-gradient(circle at 90% 80%, rgba(13, 71, 161, 0.1) 0%, transparent 30%),
                radial-gradient(circle at 50% 50%, rgba(0, 105, 217, 0.05) 0%, transparent 45%);
            z-index: -2;
        }

        .simulation-container {
            position: relative;
            width: 100%;
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            perspective: 1500px;
        }

        /* Header Styles */
        .sim-header {
            position: relative;
            z-index: 100;
            padding: 16px 24px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(18, 18, 18, 0.85);
            backdrop-filter: blur(12px);
            border-bottom: 1px solid var(--glass-border);
            height: 72px;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: 12px;
            font-weight: 800;
            font-size: 1.5rem;
            background: linear-gradient(45deg, var(--primary), #0d47a1, #01579b);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            letter-spacing: -0.5px;
        }

        .logo i {
            font-size: 2rem;
        }

        .user-profile {
            display: flex;
            align-items: center;
            gap: 12px;
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: 50px;
            padding: 8px 16px;
            backdrop-filter: blur(10px);
        }

        .xp-bar-container {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 6px;
            background: rgba(255, 255, 255, 0.1);
        }

        .xp-bar {
            height: 100%;
            background: linear-gradient(90deg, var(--primary), #0d47a1);
            border-radius: 3px;
            width: 65%;
            transition: var(--transition);
        }

        /* Main Simulation Area */
        .sim-main {
            flex: 1;
            display: flex;
            position: relative;
            overflow: hidden;
            background: linear-gradient(to bottom, #0a1122 0%, #0d1a33 100%);
        }

        .environment {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        .laminar-flow {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 5;
        }

        .hood-base {
            width: 90%;
            height: 85%;
            background: linear-gradient(145deg, #1a233a, #0f172a);
            border-radius: var(--border-radius-xl);
            box-shadow: var(--shadow-lg);
            position: relative;
            overflow: hidden;
            border: 2px solid var(--glass-border);
        }

        .hood-window {
            position: absolute;
            top: 8%;
            left: 5%;
            width: 90%;
            height: 84%;
            background: rgba(30, 41, 59, 0.92);
            border-radius: var(--border-radius-lg);
            border: 3px solid #4a6fa5;
            box-shadow: 
                inset 0 0 30px rgba(74, 111, 165, 0.3),
                0 0 20px rgba(26, 115, 232, 0.25);
            overflow: hidden;
        }

        .hood-interior {
            position: absolute;
            top: 12%;
            left: 8%;
            width: 84%;
            height: 78%;
            background: linear-gradient(145deg, #1e293b, #0f172a);
            border-radius: var(--border-radius-lg);
            border: 1px solid var(--glass-border);
            padding: 20px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            overflow: hidden;
        }

        .airflow-visualization {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 10;
        }

        .particle {
            position: absolute;
            background: rgba(255, 255, 255, 0.6);
            border-radius: 50%;
            pointer-events: none;
            opacity: 0.7;
        }

        /* Equipment and Tools */
        .equipment-zone {
            position: absolute;
            bottom: 5%;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 24px;
            z-index: 30;
            width: 90%;
            justify-content: center;
        }

        .equipment-item {
            background: var(--glass-bg);
            border: 1px solid var(--glass-border);
            border-radius: var(--border-radius-lg);
            width: 140px;
            height: 140px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            gap: 8px;
            padding: 12px;
            box-shadow: var(--shadow-md);
            backdrop-filter: blur(10px);
            cursor: grab;
            transition: var(--transition);
            position: relative;
            overflow: hidden;
            touch-action: none;
        }

        .equipment-item:active {
            cursor: grabbing;
        }

        .equipment-item.selected {
            transform: scale(1.05);
            box-shadow: 0 0 0 3px var(--primary), var(--shadow-lg);
            z-index: 40;
        }

        .equipment-item.wrong {
            animation: pulseError 0.5s ease-in-out;
            box-shadow: 0 0 0 3px var(--danger), var(--shadow-lg);
        }

        .equipment-icon {
            font-size: 2.5rem;
            margin-bottom: 8px;
        }

        .syringe-tool {
            background: linear-gradient(135deg, #e3f2fd, #bbdefb);
            color: var(--secondary);
        }

        .vial-tool {
            background: linear-gradient(135deg, #e8f5e9, #c8e6c9);
            color: #1b5e20;
        }

        .iv-bag-tool {
            background: linear-gradient(135deg, #fff8e1, #ffecb3);
            color: #5d4037;
        }

        .gloves-tool {
            background: linear-gradient(135deg, #f3e5f5, #e1bee7);
            color: #4a148c;
        }

        .label-tool {
            background: linear-gradient(135deg, #e0f7fa, #b2ebf2);
            color: #006064;
        }

        /* Interactive Elements */
        .vial {
            position: absolute;
            width: 100px;
            height: 180px;
            background: linear-gradient(145deg, #e0e0e0, #ffffff);
            border-radius: 12px 12px 40px 40px;
            bottom: 15%;
            left: 25%;
            z-index: 25;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-shadow: var(--shadow-md);
            transform-origin: center bottom;
            touch-action: none;
        }

        .vial-top {
            width: 60%;
            height: 25px;
            background: #2e7d32;
            border-radius: 8px 8px 0 0;
            margin-top: -2px;
            position: relative;
            z-index: 2;
        }

        .vial-stop {
            width: 70%;
            height: 15px;
            background: #9e9e9e;
            border-radius: 4px;
            margin-top: -5px;
            z-index: 1;
        }

        .vial-liquid {
            width: 85%;
            height: 60%;
            background: linear-gradient(to bottom, #4fc3f7, #0288d1);
            border-radius: 8px;
            margin-top: 15px;
            position: relative;
            overflow: hidden;
        }

        .vial-label {
            position: absolute;
            bottom: 25%;
            width: 70%;
            background: white;
            color: #0d47a1;
            font-weight: bold;
            text-align: center;
            padding: 4px 0;
            border-radius: 4px;
            font-size: 0.85rem;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .syringe {
            position: absolute;
            width: 60px;
            height: 200px;
            background: white;
            border-radius: 30px;
            bottom: 10%;
            right: 25%;
            z-index: 20;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-shadow: var(--shadow-md);
            touch-action: none;
            transform-origin: center bottom;
        }

        .syringe-barrel {
            width: 90%;
            height: 80%;
            background: rgba(255, 255, 255, 0.95);
            border: 2px solid #e0e0e0;
            border-radius: 20px;
            margin-top: 15px;
            position: relative;
            overflow: hidden;
        }

        .syringe-liquid {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 0%;
            background: linear-gradient(to top, #4fc3f7, #0288d1);
            border-radius: 0 0 16px 16px;
            transition: height 0.5s ease-out;
        }

        .syringe-plunger {
            width: 70%;
            height: 25%;
            background: #f5f5f5;
            border-radius: 15px 15px 0 0;
            border: 2px solid #e0e0e0;
            position: absolute;
            top: 0;
            left: 15%;
            cursor: grab;
        }

        .syringe-plunger:active {
            cursor: grabbing;
        }

        .syringe-tip {
            width: 40%;
            height: 25px;
            background: #616161;
            border-radius: 0 0 10px 10px;
            margin-top: -2px;
            position: relative;
        }

        .syringe-needle {
            position: absolute;
            bottom: -40px;
            left: 50%;
            transform: translateX(-50%);
            width: 4px;
            height: 40px;
            background: linear-gradient(to bottom, #9e9e9e, #616161);
            border-radius: 2px;
            display: none;
        }

        .syringe.selected .syringe-needle {
            display: block;
        }

        .iv-bag {
            position: absolute;
            width: 140px;
            height: 240px;
            background: rgba(255, 255, 255, 0.15);
            border: 3px solid #bbdefb;
            border-radius: 20px 20px 0 0;
            top: 15%;
            right: 15%;
            z-index: 15;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-shadow: var(--shadow-md);
            touch-action: none;
        }

        .iv-bag-top {
            width: 40%;
            height: 30px;
            background: #90a4ae;
            border-radius: 15px 15px 0 0;
            margin-top: -3px;
        }

        .iv-bag-hanger {
            width: 60%;
            height: 20px;
            background: #546e7a;
            border-radius: 10px;
            margin-top: -10px;
            position: relative;
        }

        .iv-bag-hanger::before {
            content: "";
            position: absolute;
            top: -8px;
            left: 50%;
            transform: translateX(-50%);
            width: 24px;
            height: 16px;
            background: #37474f;
            border-radius: 8px 8px 0 0;
        }

        .iv-bag-liquid {
            width: 85%;
            height: 75%;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 12px;
            margin-top: 10px;
            position: relative;
            overflow: hidden;
        }

        .iv-bag-liquid-fill {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 15%;
            background: linear-gradient(to top, rgba(79, 195, 247, 0.7), rgba(2, 136, 209, 0.4));
            border-radius: 0 0 10px 10px;
            transition: height 0.8s ease-out;
        }

        .iv-bag-tube {
            width: 12px;
            height: 60px;
            background: #546e7a;
            border-radius: 0 0 6px 6px;
            margin-top: -5px;
            position: relative;
        }

        .iv-bag-tube::after {
            content: "";
            position: absolute;
            bottom: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 20px;
            height: 15px;
            background: #37474f;
            border-radius: 0 0 8px 8px;
        }

        /* Feedback System */
        .feedback-container {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%) scale(0.8);
            opacity: 0;
            z-index: 100;
            background: white;
            color: var(--dark);
            padding: 20px 40px;
            border-radius: var(--border-radius-lg);
            font-weight: bold;
            font-size: 1.5rem;
            box-shadow: var(--shadow-lg);
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        .feedback-container.success {
            background: var(--success);
            color: white;
            transform: translate(-50%, -50%) scale(1);
            opacity: 1;
        }

        .feedback-container.error {
            background: var(--danger);
            color: white;
            transform: translate(-50%, -50%) scale(1);
            opacity: 1;
            animation: shake 0.5s;
        }

        .contamination-warning {
            position: absolute;
            top: 20%;
            left: 50%;
            transform: translateX(-50%) scale(0.9);
            opacity: 0;
            background: var(--danger);
            color: white;
            padding: 12px 30px;
            border-radius: 50px;
            font-weight: bold;
            z-index: 90;
            box-shadow: var(--shadow-md);
            transition: all 0.4s ease;
        }

        .contamination-warning.show {
            transform: translateX(-50%) scale(1);
            opacity: 1;
        }

        .glove-contamination {
            position: absolute;
            width: 120px;
            height: 150px;
            background: linear-gradient(135deg, #ffcdd2, #ef9a9a);
            border-radius: 16px;
            bottom: 15%;
            left: 15%;
            z-index: 22;
            display: none;
            animation: pulseContamination 2s infinite;
            box-shadow: 0 0 15px rgba(255, 23, 68, 0.5);
        }

        .glove-contamination.show {
            display: block;
        }

        /* Step Progress */
        .step-progress {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 12px;
            z-index: 50;
        }

        .step-indicator {
            width: 16px;
            height: 16px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.3);
            transition: var(--transition);
        }

        .step-indicator.active {
            background: var(--primary);
            transform: scale(1.3);
        }

        .step-indicator.completed {
            background: var(--success);
        }

        /* Tutorial Overlay */
        .tutorial-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 200;
            padding: 20px;
            text-align: center;
            transition: opacity 0.5s ease;
        }

        .tutorial-title {
            font-size: 2.5rem;
            margin-bottom: 24px;
            background: linear-gradient(45deg, var(--primary), #0d47a1);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            font-weight: 800;
        }

        .tutorial-description {
            font-size: 1.4rem;
            max-width: 700px;
            line-height: 1.5;
            margin-bottom: 40px;
            color: rgba(255, 255, 255, 0.85);
        }

        .start-button {
            background: linear-gradient(45deg, var(--primary), #0d47a1);
            color: white;
            border: none;
            padding: 18px 60px;
            font-size: 1.5rem;
            border-radius: 50px;
            font-weight: bold;
            cursor: pointer;
            transition: var(--transition);
            box-shadow: var(--shadow-lg);
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .start-button:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 30px rgba(13, 71, 161, 0.4);
        }

        .start-button:active {
            transform: translateY(1px);
        }

        /* Animations */
        @keyframes pulseError {
            0% { box-shadow: 0 0 0 0 rgba(255, 23, 68, 0.4); }
            70% { box-shadow: 0 0 0 12px rgba(255, 23, 68, 0); }
            100% { box-shadow: 0 0 0 0 rgba(255, 23, 68, 0); }
        }

        @keyframes shake {
            0%, 100% { transform: translate(-50%, -50%) scale(1) translateX(0); }
            20% { transform: translate(-50%, -50%) scale(1) translateX(-10px); }
            40% { transform: translate(-50%, -50%) scale(1) translateX(10px); }
            60% { transform: translate(-50%, -50%) scale(1) translateX(-5px); }
            80% { transform: translate(-50%, -50%) scale(1) translateX(5px); }
        }

        @keyframes pulseContamination {
            0% { box-shadow: 0 0 0 0 rgba(255, 23, 68, 0.3); }
            50% { box-shadow: 0 0 0 15px rgba(255, 23, 68, 0); }
            100% { box-shadow: 0 0 0 0 rgba(255, 23, 68, 0); }
        }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-10px); }
        }

        @keyframes airflow {
            0% { opacity: 0.3; transform: translateY(0) scale(0.8); }
            100% { opacity: 0; transform: translateY(-100px) scale(1.2); }
        }

        .floating {
            animation: float 3s ease-in-out infinite;
        }

        /* Responsive Adjustments */
        @media (max-width: 768px) {
            .equipment-zone {
                flex-direction: column;
                height: auto;
                bottom: 15%;
                gap: 16px;
            }
            
            .equipment-item {
                width: 120px;
                height: 120px;
            }
            
            .tutorial-title {
                font-size: 2rem;
            }
            
            .tutorial-description {
                font-size: 1.2rem;
            }
            
            .start-button {
                padding: 16px 40px;
                font-size: 1.3rem;
            }
        }
    </style>
</head>
<body>
    <div class="simulation-container">
        <header class="sim-header">
            <div class="logo">
                <i class="fas fa-mortar-pestle"></i>
                <span>ONCOSIM</span>
            </div>
            <div class="user-profile">
                <div class="user-avatar">
                    <i class="fas fa-user-md" style="font-size: 1.8rem;"></i>
                </div>
                <div class="user-info">
                    <div>Dr. Sarah Chen</div>
                    <div style="font-size: 0.85rem; opacity: 0.85;">Oncology Expert</div>
                </div>
                <div class="xp-bar-container">
                    <div class="xp-bar" style="width: 65%"></div>
                </div>
            </div>
        </header>
        
        <main class="sim-main">
            <div class="environment">
                <div class="laminar-flow">
                    <div class="hood-base">
                        <div class="hood-window"></div>
                        <div class="hood-interior">
                            <!-- Equipment will be placed here dynamically -->
                            <div class="vial floating">
                                <div class="vial-top"></div>
                                <div class="vial-stop"></div>
                                <div class="vial-liquid"></div>
                                <div class="vial-label">CYCLOPHOSPHAMIDE<br>1000mg</div>
                            </div>
                            <div class="syringe floating">
                                <div class="syringe-barrel">
                                    <div class="syringe-liquid"></div>
                                </div>
                                <div class="syringe-plunger"></div>
                                <div class="syringe-tip"></div>
                                <div class="syringe-needle"></div>
                            </div>
                            <div class="iv-bag floating">
                                <div class="iv-bag-top"></div>
                                <div class="iv-bag-hanger"></div>
                                <div class="iv-bag-liquid">
                                    <div class="iv-bag-liquid-fill"></div>
                                </div>
                                <div class="iv-bag-tube"></div>
                            </div>
                            <div class="glove-contamination"></div>
                        </div>
                    </div>
                </div>
                <div class="airflow-visualization" id="airflow"></div>
            </div>
            
            <div class="equipment-zone">
                <div class="equipment-item gloves-tool" data-tool="gloves">
                    <div class="equipment-icon">
                        <i class="fas fa-gloves"></i>
                    </div>
                    <div class="equipment-name">Sterile Gloves</div>
                </div>
                <div class="equipment-item syringe-tool" data-tool="syringe">
                    <div class="equipment-icon">
                        <i class="fas fa-syringe"></i>
                    </div>
                    <div class="equipment-name">10mL Syringe</div>
                </div>
                <div class="equipment-item vial-tool" data-tool="vial">
                    <div class="equipment-icon">
                        <i class="fas fa-vial"></i>
                    </div>
                    <div class="equipment-name">Drug Vial</div>
                </div>
                <div class="equipment-item iv-bag-tool" data-tool="ivbag">
                    <div class="equipment-icon">
                        <i class="fas fa-procedures"></i>
                    </div>
                    <div class="equipment-name">IV Bag</div>
                </div>
                <div class="equipment-item label-tool" data-tool="label">
                    <div class="equipment-icon">
                        <i class="fas fa-tag"></i>
                    </div>
                    <div class="equipment-name">Label</div>
                </div>
            </div>
            
            <div class="step-progress">
                <div class="step-indicator active"></div>
                <div class="step-indicator"></div>
                <div class="step-indicator"></div>
                <div class="step-indicator"></div>
                <div class="step-indicator"></div>
            </div>
            
            <div class="feedback-container" id="feedback"></div>
            <div class="contamination-warning" id="contamination">Contamination detected! Aseptic technique broken.</div>
        </main>
        
        <div class="tutorial-overlay" id="tutorial">
            <h1 class="tutorial-title">Chemotherapy Preparation Simulator</h1>
            <p class="tutorial-description">
                Welcome to OncoSim, the immersive training platform for IV chemotherapy preparation. 
                This simulation replicates real-world aseptic technique in a biological safety cabinet. 
                Follow the steps carefully - every action matters when handling hazardous drugs.
            </p>
            <p class="tutorial-description" style="margin-top: 15px; font-weight: 500;">
                <i class="fas fa-hand-point-right" style="margin-right: 10px;"></i>
                Use touch gestures to interact with equipment<br>
                <i class="fas fa-hand-point-right" style="margin-right: 10px; margin-top: 8px;"></i>
                Maintain sterile technique to avoid contamination<br>
                <i class="fas fa-hand-point-right" style="margin-right: 10px; margin-top: 8px;"></i>
                Complete all steps accurately to prepare the medication
            </p>
            <button class="start-button" id="start-sim">
                <i class="fas fa-play-circle"></i> Begin Training
            </button>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // DOM Elements
            const tutorialOverlay = document.getElementById('tutorial');
            const startButton = document.getElementById('start-sim');
            const feedbackElement = document.getElementById('feedback');
            const contaminationWarning = document.getElementById('contamination');
            const syringe = document.querySelector('.syringe');
            const syringePlunger = document.querySelector('.syringe-plunger');
            const syringeLiquid = document.querySelector('.syringe-liquid');
            const ivBagLiquid = document.querySelector('.iv-bag-liquid-fill');
            const vial = document.querySelector('.vial');
            const gloveContamination = document.querySelector('.glove-contamination');
            const equipmentItems = document.querySelectorAll('.equipment-item');
            const airflowContainer = document.getElementById('airflow');
            
            // Simulation state
            let simulationStarted = false;
            let currentStep = 0;
            let syringeSelected = false;
            let plungerPosition = 0;
            let contaminationDetected = false;
            let drugWithdrawn = false;
            
            // Initialize airflow particles
            function initAirflow() {
                for (let i = 0; i < 40; i++) {
                    const particle = document.createElement('div');
                    particle.className = 'particle';
                    particle.style.width = `${Math.random() * 8 + 4}px`;
                    particle.style.height = particle.style.width;
                    particle.style.left = `${Math.random() * 100}%`;
                    particle.style.top = `${Math.random() * 100}%`;
                    particle.style.animation = `airflow ${Math.random() * 3 + 2}s linear infinite`;
                    particle.style.animationDelay = `${Math.random() * 2}s`;
                    airflowContainer.appendChild(particle);
                }
            }
            
            // Show feedback message
            function showFeedback(message, type) {
                feedbackElement.textContent = message;
                feedbackElement.className = 'feedback-container';
                feedbackElement.classList.add(type);
                
                // Trigger haptic feedback
                if (navigator.vibrate && type === 'error') {
                    navigator.vibrate([100, 50, 100]);
                } else if (navigator.vibrate && type === 'success') {
                    navigator.vibrate(50);
                }
                
                // Play sound effect
                if (type === 'error') {
                    playSound('error');
                } else if (type === 'success') {
                    playSound('success');
                }
                
                setTimeout(() => {
                    feedbackElement.className = 'feedback-container';
                }, 2500);
            }
            
            // Play sound effects (simulated)
            function playSound(type) {
                // In a real app, this would play actual sound files
                console.log(`Playing sound: ${type}`);
            }
            
            // Detect contamination
            function detectContamination() {
                if (!contaminationDetected) {
                    contaminationDetected = true;
                    contaminationWarning.classList.add('show');
                    gloveContamination.classList.add('show');
                    showFeedback('Contamination detected! Aseptic technique broken.', 'error');
                    
                    // Reset after delay
                    setTimeout(() => {
                        contaminationDetected = false;
                        contaminationWarning.classList.remove('show');
                        gloveContamination.classList.remove('show');
                    }, 5000);
                }
            }
            
            // Initialize equipment drag interactions
            function initEquipmentDrag() {
                equipmentItems.forEach(item => {
                    item.addEventListener('touchstart', handleTouchStart);
                    item.addEventListener('touchend', handleTouchEnd);
                    item.addEventListener('mousedown', handleTouchStart);
                    item.addEventListener('mouseup', handleTouchEnd);
                });
            }
            
            function handleTouchStart(e) {
                e.preventDefault();
                const tool = this.getAttribute('data-tool');
                
                // Deselect all items
                equipmentItems.forEach(item => item.classList.remove('selected', 'wrong'));
                
                // Select current item
                this.classList.add('selected');
                
                // Handle specific tool actions
                if (tool === 'syringe' && !syringeSelected) {
                    syringeSelected = true;
                    syringe.classList.add('selected');
                    showFeedback('Syringe selected. Tap vial to withdraw medication.', 'success');
                } else if (tool === 'gloves' && currentStep === 0) {
                    currentStep = 1;
                    updateStepProgress();
                    showFeedback('Sterile gloves applied. Remember to maintain aseptic technique!', 'success');
                } else if (tool === 'gloves' && currentStep > 0) {
                    this.classList.add('wrong');
                    showFeedback('Gloves already applied. Proceed with medication preparation.', 'error');
                } else if (tool !== 'syringe' && syringeSelected) {
                    this.classList.add('wrong');
                    showFeedback('Complete drug withdrawal before selecting another tool.', 'error');
                }
            }
            
            function handleTouchEnd(e) {
                // No-op for now, could add additional logic here
            }
            
            // Initialize syringe plunger interaction
            function initSyringeInteraction() {
                let isDragging = false;
                let startY = 0;
                let startPlungerPos = 0;
                
                syringePlunger.addEventListener('touchstart', handlePlungerStart, { passive: false });
                syringePlunger.addEventListener('touchmove', handlePlungerMove, { passive: false });
                syringePlunger.addEventListener('touchend', handlePlungerEnd);
                
                syringePlunger.addEventListener('mousedown', handlePlungerStart);
                document.addEventListener('mousemove', handlePlungerMove);
                document.addEventListener('mouseup', handlePlungerEnd);
                
                function handlePlungerStart(e) {
                    e.preventDefault();
                    if (!syringeSelected) return;
                    
                    isDragging = true;
                    startY = e.type === 'touchstart' ? e.touches[0].clientY : e.clientY;
                    startPlungerPos = plungerPosition;
                    
                    syringePlunger.style.cursor = 'grabbing';
                    playSound('click');
                }
                
                function handlePlungerMove(e) {
                    if (!isDragging) return;
                    e.preventDefault();
                    
                    const currentY = e.type === 'touchmove' ? e.touches[0].clientY : e.clientY;
                    const deltaY = currentY - startY;
                    
                    // Calculate new plunger position (0 to 100%)
                    let newPos = startPlungerPos + (deltaY / 2);
                    newPos = Math.max(0, Math.min(100, newPos));
                    
                    // Update plunger position and liquid fill
                    plungerPosition = newPos;
                    syringePlunger.style.top = `${newPos}%`;
                    syringeLiquid.style.height = `${newPos}%`;
                    
                    // Detect if drug is being withdrawn
                    if (newPos > 20 && !drugWithdrawn) {
                        drugWithdrawn = true;
                        showFeedback('Drug successfully withdrawn. Inject into IV bag.', 'success');
                    }
                }
                
                function handlePlungerEnd() {
                    if (!isDragging) return;
                    
                    isDragging = false;
                    syringePlunger.style.cursor = 'grab';
                    
                    // Validate withdrawal amount
                    if (plungerPosition < 15) {
                        showFeedback('Insufficient drug withdrawn. Withdraw at least 15% of syringe volume.', 'error');
                        plungerPosition = 0;
                        syringeLiquid.style.height = '0%';
                        syringePlunger.style.top = '0%';
                        drugWithdrawn = false;
                    }
                }
            }
            
            // Initialize vial interaction
            function initVialInteraction() {
                vial.addEventListener('touchend', handleVialTap);
                vial.addEventListener('mouseup', handleVialTap);
            }
            
            function handleVialTap(e) {
                if (syringeSelected && !drugWithdrawn && currentStep >= 1) {
                    // Animate needle piercing
                    gsap.to('.syringe', {
                        duration: 0.4,
                        x: -150,
                        y: -50,
                        rotation: -5,
                        ease: "power2.out"
                    });
                    
                    gsap.to('.syringe-needle', {
                        duration: 0.3,
                        display: 'block',
                        ease: "power1.out"
                    });
                    
                    // Show feedback
                    showFeedback('Needle inserted into vial. Drag plunger to withdraw medication.', 'success');
                    
                    // After delay, reset syringe position but keep needle visible
                    setTimeout(() => {
                        gsap.to('.syringe', {
                            duration: 0.5,
                            x: 0,
                            y: 0,
                            rotation: 0,
                            ease: "power2.out"
                        });
                    }, 800);
                } else if (!syringeSelected) {
                    showFeedback('Select syringe first before accessing vial.', 'error');
                } else if (drugWithdrawn) {
                    showFeedback('Drug already withdrawn. Proceed to IV bag.', 'error');
                }
            }
            
            // Initialize IV bag interaction
            function initIVBagInteraction() {
                document.querySelector('.iv-bag').addEventListener('touchend', handleIVBagTap);
                document.querySelector('.iv-bag').addEventListener('mouseup', handleIVBagTap);
            }
            
            function handleIVBagTap(e) {
                if (drugWithdrawn && syringeSelected) {
                    // Animate injection
                    gsap.to('.syringe', {
                        duration: 0.5,
                        x: 100,
                        y: -100,
                        rotation: 10,
                        ease: "power2.out"
                    });
                    
                    // Animate fluid transfer
                    setTimeout(() => {
                        gsap.to(ivBagLiquid, {
                            duration: 1.5,
                            height: '45%',
                            ease: "power2.out"
                        });
                        
                        gsap.to(syringeLiquid, {
                            duration: 1.0,
                            height: '0%',
                            ease: "power2.in"
                        });
                        
                        // Reset syringe state
                        setTimeout(() => {
                            plungerPosition = 0;
                            drugWithdrawn = false;
                            syringeSelected = false;
                            syringe.classList.remove('selected');
                            equipmentItems.forEach(item => item.classList.remove('selected'));
                            
                            // Advance to next step
                            currentStep = 2;
                            updateStepProgress();
                            
                            showFeedback('Medication successfully transferred to IV bag. Apply label to complete.', 'success');
                        }, 1000);
                    }, 600);
                } else if (!drugWithdrawed) {
                    showFeedback('Withdraw medication from vial before injecting into IV bag.', 'error');
                }
            }
            
            // Update step progress indicators
            function updateStepProgress() {
                document.querySelectorAll('.step-indicator').forEach((indicator, index) => {
                    if (index < currentStep) {
                        indicator.classList.add('completed');
                        indicator.classList.remove('active');
                    } else if (index === currentStep) {
                        indicator.classList.add('active');
                        indicator.classList.remove('completed');
                    } else {
                        indicator.classList.remove('active', 'completed');
                    }
                });
            }
            
            // Device motion detection for contamination events
            function initDeviceMotion() {
                if (window.DeviceMotionEvent) {
                    window.addEventListener('devicemotion', (event) => {
                        if (!simulationStarted || contaminationDetected) return;
                        
                        const acceleration = event.accelerationIncludingGravity;
                        const threshold = 15; // Adjust sensitivity
                        
                        if (Math.abs(acceleration.x) > threshold || 
                            Math.abs(acceleration.y) > threshold || 
                            Math.abs(acceleration.z) > threshold) {
                            detectContamination();
                        }
                    });
                }
            }
            
            // Start simulation
            startButton.addEventListener('click', () => {
                tutorialOverlay.style.opacity = '0';
                setTimeout(() => {
                    tutorialOverlay.style.display = 'none';
                }, 500);
                
                simulationStarted = true;
                initAirflow();
                initEquipmentDrag();
                initSyringeInteraction();
                initVialInteraction();
                initIVBagInteraction();
                initDeviceMotion();
                
                // Initial tutorial hint
                setTimeout(() => {
                    showFeedback('Tap sterile gloves to begin preparation', 'success');
                }, 1000);
            });
            
            // Initialize step progress
            updateStepProgress();
            
            // Add touch event to document to prevent default behavior on touch devices
            document.addEventListener('touchmove', (e) => {
                if (!e.target.closest('.syringe-plunger') && 
                    !e.target.closest('.equipment-item')) {
                    e.preventDefault();
                }
            }, { passive: false });
        });
    </script>
</body>
</html>
