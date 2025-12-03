import React, { useState, useRef, useEffect } from 'react';
import { Upload, Download, Image as ImageIcon, Move, ZoomIn, ZoomOut, RefreshCw, Smartphone, Heart, Award } from 'lucide-react';

export default function App() {
  const canvasRef = useRef(null);
  const [uploadedImage, setUploadedImage] = useState(null);
  const [selectedFrame, setSelectedFrame] = useState(0);
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [scale, setScale] = useState(1);
  const [isDragging, setIsDragging] = useState(false);
  const [dragStart, setDragStart] = useState({ x: 0, y: 0 });

  // ക്യാമ്പയിൻ ഫ്രെയിമുകളുടെ വിവരങ്ങൾ (Election only)
  const frames = [
    {
      id: 1,
      name: "Election",
      color: "#2563eb",
      title: "VOTE FOR CHANGE",
      footer: "Your Vote, Your Future • Election 2025",
      icon: "Vote"
    }
  ];

  // ചിത്രം അപ്‌ലോഡ് ചെയ്യുമ്പോൾ പ്രവർത്തിക്കുന്ന ഫംഗ്‌ഷൻ
  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        const img = new Image();
        img.onload = () => {
          setUploadedImage(img);
          setPosition({ x: 0, y: 0 }); // Reset position
          setScale(1); // Reset scale
        };
        img.src = event.target.result;
      };
      reader.readAsDataURL(file);
    }
  };

  // Canvas-ൽ ചിത്രം വരയ്ക്കുന്ന പ്രധാന ഫംഗ്‌ഷൻ
  const drawCanvas = () => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const width = 1000; // Output quality width (Square for WhatsApp DP)
    const height = 1000; // Output quality height (Square for WhatsApp DP)

    // Canvas size set ചെയ്യുക
    canvas.width = width;
    canvas.height = height;

    // --- CIRCULAR CLIPPING START ---
    // വട്ടത്തിൽ ക്രോപ്പ് ചെയ്യാനുള്ള കോഡ്
    ctx.beginPath();
    ctx.arc(width / 2, height / 2, width / 2, 0, Math.PI * 2);
    ctx.closePath();
    ctx.clip(); // താഴെയുള്ള എല്ലാം ഈ വട്ടത്തിനുള്ളിൽ മാത്രമേ വരയ്ക്കൂ
    // --- CIRCULAR CLIPPING END ---

    // 1. പശ്ചാത്തലം ക്ലിയർ ചെയ്യുക (Clear Canvas)
    ctx.fillStyle = "#f3f4f6";
    ctx.fillRect(0, 0, width, height);

    // 2. അപ്‌ലോഡ് ചെയ്ത ചിത്രം വരയ്ക്കുക (Draw User Image)
    if (uploadedImage) {
      ctx.save();
      // മധ്യഭാഗത്തേക്ക് മാറ്റുന്നു
      ctx.translate(width / 2 + position.x, height / 2 + position.y);
      ctx.scale(scale, scale);
      // ചിത്രം നടുവിൽ വരുന്ന രീതിയിൽ വരയ്ക്കുന്നു
      ctx.drawImage(uploadedImage, -uploadedImage.width / 2, -uploadedImage.height / 2);
      ctx.restore();
    } else {
      // ചിത്രം ഇല്ലെങ്കിൽ കാണിക്കേണ്ട ടെക്സ്റ്റ്
      ctx.fillStyle = "#9ca3af";
      ctx.font = "bold 30px sans-serif";
      ctx.textAlign = "center";
      ctx.fillText("Upload a photo to start", width / 2, height / 2);
    }

    // 3. ഫ്രെയിം ഓവർലേ വരയ്ക്കുക (Draw Frame Overlay)
    drawOverlayFrame(ctx, width, height, frames[selectedFrame]);
  };

  // ഫ്രെയിം ഡിസൈൻ ചെയ്യുന്ന ഫംഗ്‌ഷൻ (Simulating a PNG overlay)
  const drawOverlayFrame = (ctx, w, h, frame) => {
    const borderSize = 40;
    
    // Gradient Overlay at bottom (Circular style)
    const gradient = ctx.createLinearGradient(0, h/2, 0, h);
    gradient.addColorStop(0, "transparent");
    gradient.addColorStop(0.6, frame.color); 
    gradient.addColorStop(1, "black");
    
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, w, h); // Fill rect is fine because we have a circular clip applied globally

    // Circular Border Frame
    ctx.beginPath();
    ctx.arc(w / 2, h / 2, (w / 2) - (borderSize / 2), 0, Math.PI * 2);
    ctx.lineWidth = borderSize;
    ctx.strokeStyle = frame.color;
    ctx.stroke();

    // Text Content
    ctx.fillStyle = "white";
    ctx.textAlign = "center";
    
    // Top Tag (#SocialWaves) - Curved Text or Simple placement
    // Since it's round, let's place it a bit lower
    ctx.save();
    ctx.fillStyle = "white";
    ctx.font = "bold 30px sans-serif";
    ctx.shadowColor = "black";
    ctx.shadowBlur = 4;
    ctx.fillText("#SocialWaves", w / 2, h - 100);
    ctx.restore();
  };

  // ചിത്രം വരയ്ക്കാനുള്ള useEffect
  useEffect(() => {
    drawCanvas();
  }, [uploadedImage, selectedFrame, position, scale]);

  // മൗസ് ഉപയോഗിച്ച് ചിത്രം നീക്കാനുള്ള ലോജിക് (Pan Logic)
  const handleMouseDown = (e) => {
    setIsDragging(true);
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;
    setDragStart({ x: clientX - position.x, y: clientY - position.y });
  };

  const handleMouseMove = (e) => {
    if (!isDragging) return;
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;
    setPosition({
      x: clientX - dragStart.x,
      y: clientY - dragStart.y
    });
  };

  const handleMouseUp = () => {
    setIsDragging(false);
  };

  // ചിത്രം ഡൗൺലോഡ് ചെയ്യാനുള്ള ഫംഗ്‌ഷൻ
  const downloadPoster = () => {
    const canvas = canvasRef.current;
    const link = document.createElement('a');
    link.download = 'my-social-campaign.png';
    link.href = canvas.toDataURL('image/png');
    link.click();
  };

  return (
    <div className="min-h-screen bg-gray-50 text-gray-800 font-sans">
      {/* Header */}
      <header className="bg-white shadow-sm p-4 sticky top-0 z-10">
        <div className="max-w-4xl mx-auto flex justify-between items-center">
          <div className="flex items-center gap-2">
            <div className="bg-indigo-600 p-2 rounded-lg text-white">
              <Award size={24} />
            </div>
            <h1 className="text-xl font-bold text-gray-900">SocialWaves Maker</h1>
          </div>
          <button 
            onClick={downloadPoster}
            disabled={!uploadedImage}
            className={`flex items-center gap-2 px-4 py-2 rounded-full font-medium transition-colors ${
              uploadedImage 
              ? "bg-indigo-600 text-white hover:bg-indigo-700 shadow-md" 
              : "bg-gray-200 text-gray-400 cursor-not-allowed"
            }`}
          >
            <Download size={18} />
            <span className="hidden sm:inline">Download Poster</span>
          </button>
        </div>
      </header>

      <main className="max-w-4xl mx-auto p-4 md:p-8">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
          
          {/* Left Column: Controls */}
          <div className="space-y-6 order-2 md:order-1">
            
            {/* 1. Upload Section */}
            <div className="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
              <h2 className="text-lg font-semibold mb-4 flex items-center gap-2">
                <ImageIcon className="text-indigo-500" />
                1. Upload Photo
              </h2>
              <label className="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed border-indigo-200 rounded-xl cursor-pointer bg-indigo-50 hover:bg-indigo-100 transition-colors">
                <div className="flex flex-col items-center justify-center pt-5 pb-6">
                  <Upload className="w-8 h-8 mb-2 text-indigo-500" />
                  <p className="text-sm text-gray-500 font-medium">Click to upload image</p>
                </div>
                <input type="file" className="hidden" accept="image/*" onChange={handleImageUpload} />
              </label>
            </div>

            {/* 2. Controls Section */}
            {uploadedImage && (
              <div className="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
                <h2 className="text-lg font-semibold mb-4 flex items-center gap-2">
                  <Move className="text-indigo-500" />
                  2. Adjust Image
                </h2>
                
                <div className="space-y-4">
                  <div className="flex items-center gap-4">
                    <ZoomOut size={20} className="text-gray-500" />
                    <input 
                      type="range" 
                      min="0.1" 
                      max="3" 
                      step="0.1" 
                      value={scale} 
                      onChange={(e) => setScale(parseFloat(e.target.value))}
                      className="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-indigo-600"
                    />
                    <ZoomIn size={20} className="text-gray-500" />
                  </div>
                  <p className="text-xs text-center text-gray-400">
                    Drag the image on the preview to move it
                  </p>
                  <button 
                    onClick={() => { setPosition({x:0, y:0}); setScale(1); }}
                    className="w-full py-2 text-sm text-indigo-600 bg-indigo-50 rounded-lg hover:bg-indigo-100 flex items-center justify-center gap-2"
                  >
                    <RefreshCw size={14} /> Reset Position
                  </button>
                </div>
              </div>
            )}

            {/* 3. Frame Selection */}
            <div className="bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
              <h2 className="text-lg font-semibold mb-4 flex items-center gap-2">
                <Smartphone className="text-indigo-500" />
                3. Active Campaign
              </h2>
              <div className="grid grid-cols-3 gap-3">
                {frames.map((frame, index) => (
                  <button
                    key={frame.id}
                    onClick={() => setSelectedFrame(index)}
                    className={`p-2 rounded-xl border-2 transition-all ${
                      selectedFrame === index 
                      ? "border-indigo-600 bg-indigo-50" 
                      : "border-gray-200 hover:border-gray-300"
                    }`}
                  >
                    <div 
                      className="w-full h-16 rounded-md mb-2" 
                      style={{ backgroundColor: frame.color }}
                    ></div>
                    <p className="text-xs font-medium text-center truncate">{frame.name}</p>
                  </button>
                ))}
              </div>
            </div>
          </div>

          {/* Right Column: Preview */}
          <div className="order-1 md:order-2">
            <div className="sticky top-24">
              <div className="bg-white p-4 rounded-3xl shadow-lg border border-gray-200">
                <div className="relative w-full aspect-square bg-gray-100 rounded-xl overflow-hidden cursor-move touch-none">
                  <canvas
                    ref={canvasRef}
                    className="w-full h-full object-contain"
                    onMouseDown={handleMouseDown}
                    onMouseMove={handleMouseMove}
                    onMouseUp={handleMouseUp}
                    onMouseLeave={handleMouseUp}
                    onTouchStart={handleMouseDown}
                    onTouchMove={handleMouseMove}
                    onTouchEnd={handleMouseUp}
                  />
                  
                  {!uploadedImage && (
                    <div className="absolute inset-0 flex items-center justify-center pointer-events-none">
                      <div className="text-center p-6 opacity-40">
                         <ImageIcon size={48} className="mx-auto mb-2" />
                         <p>Preview will appear here</p>
                      </div>
                    </div>
                  )}
                </div>
                <div className="mt-4 text-center">
                  <p className="text-sm font-medium text-gray-500">
                     Generated Poster Preview
                  </p>
                </div>
              </div>
            </div>
          </div>

        </div>
      </main>
    </div>
  );
}
