// EditProX Full App: Trim + Music + Text + Filters + Export

import 'dart:io'; import 'package:flutter/material.dart'; import 'package:file_picker/file_picker.dart'; import 'package:video_player/video_player.dart'; import 'package:ffmpeg_kit_flutter/ffmpeg_kit.dart'; import 'package:path_provider/path_provider.dart'; import 'package:path/path.dart' as path;

void main() => runApp(EditProX());

class EditProX extends StatelessWidget { @override Widget build(BuildContext context) => MaterialApp( debugShowCheckedModeBanner: false, title: 'EditProX', theme: ThemeData.dark(), home: HomeScreen(), ); }

class HomeScreen extends StatelessWidget { @override Widget build(BuildContext context) => Scaffold( appBar: AppBar(title: Text("EditProX Video Editor")), body: Center( child: Column( mainAxisAlignment: MainAxisAlignment.center, children: [ FeatureButton("🎬 Trim Video", () => Navigator.push(context, MaterialPageRoute(builder: () => TrimVideoScreen()))), FeatureButton("🎵 Add Music", () => Navigator.push(context, MaterialPageRoute(builder: () => AddMusicScreen()))), FeatureButton("🔤 Add Text", () => Navigator.push(context, MaterialPageRoute(builder: () => AddTextScreen()))), FeatureButton("🎨 Filters", () => Navigator.push(context, MaterialPageRoute(builder: () => FilterScreen()))), FeatureButton("📤 Export Video", () => Navigator.push(context, MaterialPageRoute(builder: (_) => ExportScreen()))), ], ), ), ); }

class FeatureButton extends StatelessWidget { final String title; final VoidCallback onTap; FeatureButton(this.title, this.onTap); @override Widget build(BuildContext context) => Padding( padding: const EdgeInsets.symmetric(vertical: 10.0), child: ElevatedButton( onPressed: onTap, style: ElevatedButton.styleFrom(padding: EdgeInsets.symmetric(horizontal: 40, vertical: 15)), child: Text(title, style: TextStyle(fontSize: 18)), ), ); }

// === Existing Screens === class TrimVideoScreen extends StatelessWidget { /* Already Implemented / Widget build(_) => Scaffold(body: Center(child: Text('Trim Module'))); } class AddMusicScreen extends StatelessWidget { / Already Implemented / Widget build(_) => Scaffold(body: Center(child: Text('Music Module'))); } class AddTextScreen extends StatelessWidget { / Already Implemented */ Widget build(_) => Scaffold(body: Center(child: Text('Text Module'))); }

// === New: Filter Module === class FilterScreen extends StatefulWidget { @override _FilterScreenState createState() => _FilterScreenState(); }

class _FilterScreenState extends State<FilterScreen> { String? _videoPath; bool _isProcessing = false; String? _selectedFilter = 'grayscale';

final Map<String, String> filters = { 'grayscale': 'hue=s=0', 'vintage': 'curves=vintage', 'bright': 'eq=brightness=0.06', 'contrast': 'eq=contrast=1.5', };

Future<void> _pickVideo() async { final result = await FilePicker.platform.pickFiles(type: FileType.video); if (result != null && result.files.single.path != null) { setState(() => _videoPath = result.files.single.path); } }

Future<void> _applyFilter() async { if (_videoPath == null || _selectedFilter == null) return; setState(() => isProcessing = true); final outDir = await getTemporaryDirectory(); final outPath = path.join(outDir.path, "filtered${DateTime.now().millisecondsSinceEpoch}.mp4"); final command = "-i '$_videoPath' -vf ${filters[_selectedFilter]} -c:a copy '$outPath'"; await FFmpegKit.execute(command); setState(() => _isProcessing = false); ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text("Filtered video saved at: $outPath"))); }

@override Widget build(BuildContext context) => Scaffold( appBar: AppBar(title: Text("Apply Filters")), body: Padding( padding: const EdgeInsets.all(16.0), child: Column( children: [ ElevatedButton(onPressed: _pickVideo, child: Text("📁 Pick Video")), if (_videoPath != null) ...[ DropdownButton<String>( value: _selectedFilter, items: filters.keys.map((filter) => DropdownMenuItem(value: filter, child: Text(filter))).toList(), onChanged: (val) => setState(() => _selectedFilter = val), ), ElevatedButton( onPressed: _isProcessing ? null : _applyFilter, child: _isProcessing ? CircularProgressIndicator() : Text("🎨 Apply Filter"), ) ], ], ), ), ); }

// === New: Export Screen === class ExportScreen extends StatefulWidget { @override _ExportScreenState createState() => _ExportScreenState(); }

class _ExportScreenState extends State<ExportScreen> { String? _videoPath; String _resolution = '720p'; bool _includeWatermark = true; bool _isProcessing = false;

final resolutions = { '720p': '1280x720', '1080p': '1920x1080', '4K': '3840x2160', };

Future<void> _pickVideo() async { final result = await FilePicker.platform.pickFiles(type: FileType.video); if (result != null && result.files.single.path != null) { setState(() => _videoPath = result.files.single.path); } }

Future<void> _exportVideo() async { if (_videoPath == null) return; setState(() => isProcessing = true); final outDir = await getTemporaryDirectory(); final outPath = path.join(outDir.path, "export${DateTime.now().millisecondsSinceEpoch}.mp4"); final scale = resolutions[_resolution]; final watermark = _includeWatermark ? ",drawtext=text='EditProX':fontcolor=white:x=10:y=10" : ""; final command = "-i '$_videoPath' -vf scale=$scale$watermark -c:a copy '$outPath'"; await FFmpegKit.execute(command); setState(() => _isProcessing = false); ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text("Exported video saved at: $outPath"))); }

@override Widget build(BuildContext context) => Scaffold( appBar: AppBar(title: Text("Export Video")), body: Padding( padding: const EdgeInsets.all(16.0), child: Column( children: [ ElevatedButton(onPressed: _pickVideo, child: Text("📁 Pick Video")), if (_videoPath != null) ...[ DropdownButton<String>( value: _resolution, items: resolutions.keys.map((r) => DropdownMenuItem(value: r, child: Text(r))).toList(), onChanged: (val) => setState(() => _resolution = val!), ), SwitchListTile( title: Text("Include Watermark"), value: _includeWatermark, onChanged: (val) => setState(() => _includeWatermark = val), ), ElevatedButton( onPressed: _isProcessing ? null : _exportVideo, child: _isProcessing ? CircularProgressIndicator() : Text("📤 Export Now"), ) ] ], ), ), ); } // End of File

