// ============================================
// CANCHA - RED SOCIAL DE LA VIDA
// Código completo en un solo archivo
// ============================================

// ============================================
// SECCIÓN 1: CONFIGURACIÓN Y DEPENDENCIAS
// ============================================

const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const multer = require('multer');
const path = require('path');
const { createServer } = require('http');
const { Server } = require('socket.io');
const cloudinary = require('cloudinary').v2;
const { CloudinaryStorage } = require('multer-storage-cloudinary');

// Configuración Cloudinary (para imágenes/videos)
cloudinary.config({
  cloud_name: process.env.CLOUDINARY_NAME || 'demo',
  api_key: process.env.CLOUDINARY_KEY || 'demo',
  api_secret: process.env.CLOUDINARY_SECRET || 'demo'
});

// ============================================
// SECCIÓN 2: MODELOS DE BASE DE DATOS (MongoDB)
// ============================================

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  perfil: {
    nombre: String,
    username: { type: String, unique: true, sparse: true },
    foto: { type: String, default: 'https://res.cloudinary.com/demo/image/upload/v1/default-avatar' },
    biografia: { type: String, default: '' },
    ubicacion: String,
    fechaNacimiento: Date,
    genero: String,
    intereses: [String],
    profesion: String,
    sitioWeb: String
  },
  privacidad: {
    cuentaPrivada: { type: Boolean, default: false },
    mostrarActividad: { type: Boolean, default: true },
    permitirMensajes: { type: String, default: 'todos' } // todos, seguidores, nadie
  },
  seguidores: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  seguidos: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  solicitudesSeguimiento: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  bloqueados: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  verificado: { type: Boolean, default: false },
  ultimaActividad: { type: Date, default: Date.now },
  createdAt: { type: Date, default: Date.now }
});

const postSchema = new mongoose.Schema({
  autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  tipo: { type: String, enum: ['foto', 'video', 'carrusel', 'texto'], required: true },
  contenido: [{
    url: String,
    descripcion: String,
    etiquetas: [{ 
      usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
      posicion: { x: Number, y: Number }
    }]
  }],
  caption: String,
  hashtags: [String],
  ubicacion: String,
  likes: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, timestamp: Date }],
  comentarios: [{
    autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    texto: String,
    likes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    respuestas: [{
      autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
      texto: String,
      likes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
      createdAt: { type: Date, default: Date.now }
    }],
    createdAt: { type: Date, default: Date.now }
  }],
  guardados: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  compartidos: { type: Number, default: 0 },
  ocultarLikes: { type: Boolean, default: false },
  desactivarComentarios: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
});

const storySchema = new mongoose.Schema({
  autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  tipo: { type: String, enum: ['imagen', 'video', 'texto', 'encuesta'], required: true },
  contenido: {
    url: String,
    texto: String,
    colorFondo: String,
    fuente: String,
    opciones: [{ texto: String, votos: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }] }]
  },
  filtros: {
    musica: {
      titulo: String,
      artista: String,
      tiempoInicio: Number
    },
    ubicacion: String,
    menciones: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    hashtags: [String]
  },
  vistas: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, timestamp: Date }],
  reacciones: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, tipo: String, timestamp: Date }],
  destacada: { type: Boolean, default: false },
  categoriaDestacada: String,
  expiraAt: { type: Date, default: () => new Date(Date.now() + 24 * 60 * 60 * 1000) },
  createdAt: { type: Date, default: Date.now }
});

const conversationSchema = new mongoose.Schema({
  participantes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  tipo: { type: String, enum: ['individual', 'grupo'], default: 'individual' },
  nombre: String, // para grupos
  foto: String,   // para grupos
  mensajes: [{
    autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    tipo: { type: String, enum: ['texto', 'imagen', 'video', 'audio', 'sticker', 'ubicacion', 'publicacion'] },
    contenido: String,
    publicacionRef: { type: mongoose.Schema.Types.ObjectId, ref: 'Post' },
    reacciones: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, emoji: String }],
    respondidoA: { type: mongoose.Schema.Types.ObjectId },
    eliminado: { type: Boolean, default: false },
    vistoPor: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, timestamp: Date }],
    createdAt: { type: Date, default: Date.now }
  }],
  notas: [{
    autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    texto: String,
    musica: String,
    expiraAt: Date,
    createdAt: { type: Date, default: Date.now }
  }],
  ultimoMensaje: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  silenciado: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  fijado: { type: Boolean, default: false },
  updatedAt: { type: Date, default: Date.now }
});

const notificationSchema = new mongoose.Schema({
  destinatario: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  tipo: { type: String, enum: ['like', 'comentario', 'seguimiento', 'solicitud', 'mencion', 'mensaje', 'respuesta', 'compartido'] },
  emisor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  referencia: {
    tipo: String, // publicacion, comentario, historia, mensaje, usuario
    id: mongoose.Schema.Types.ObjectId
  },
  mensaje: String,
  leida: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);
const Post = mongoose.model('Post', postSchema);
const Story = mongoose.model('Story', storySchema);
const Conversation = mongoose.model('Conversation', conversationSchema);
const Notification = mongoose.model('Notification', notificationSchema);

// ============================================
// SECCIÓN 3: MIDDLEWARE Y CONFIGURACIÓN
// ============================================

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: '*' } });

app.use(cors());
app.use(express.json());
app.use(express.static('public'));

// Middleware de autenticación
const authMiddleware = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'Token requerido' });
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET || 'cancha-secret-key');
    req.userId = decoded.userId;
    req.user = await User.findById(decoded.userId);
    next();
  } catch (error) {
    res.status(401).json({ error: 'Token inválido' });
  }
};

// Configuración Multer para subida de archivos
const storage = new CloudinaryStorage({
  cloudinary: cloudinary,
  params: {
    folder: 'cancha',
    allowed_formats: ['jpg', 'png', 'mp4', 'mov'],
    transformation: [{ width: 1080, height: 1080, crop: 'limit' }]
  }
});
const upload = multer({ storage });

// ============================================
// SECCIÓN 4: RUTAS DE AUTENTICACIÓN
// ============================================

// Registro
app.post('/api/auth/register', async (req, res) => {
  try {
    const { email, password, nombre, username } = req.body;
    
    const existingUser = await User.findOne({ $or: [{ email }, { 'perfil.username': username }] });
    if (existingUser) {
      return res.status(400).json({ error: 'Email o username ya existe' });
    }
    
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({
      email,
      password: hashedPassword,
      perfil: { nombre, username }
    });
    
    await user.save();
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET || 'cancha-secret-key', { expiresIn: '7d' });
    
    res.status(201).json({ token, user: { id: user._id, email, perfil: user.perfil } });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Login
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    
    if (!user || !await bcrypt.compare(password, user.password)) {
      return res.status(401).json({ error: 'Credenciales inválidas' });
    }
    
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET || 'cancha-secret-key', { expiresIn: '7d' });
    user.ultimaActividad = new Date();
    await user.save();
    
    res.json({ token, user: { id: user._id, email, perfil: user.perfil } });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 5: RUTAS DE USUARIO
// ============================================

// Obtener perfil
app.get('/api/users/:username', authMiddleware, async (req, res) => {
  try {
    const user = await User.findOne({ 'perfil.username': req.params.username })
      .populate('seguidores', 'perfil.username perfil.foto')
      .populate('seguidos', 'perfil.username perfil.foto');
    
    if (!user) return res.status(404).json({ error: 'Usuario no encontrado' });
    
    const esSeguidor = user.seguidores.some(s => s._id.toString() === req.userId);
    const solicitudEnviada = user.solicitudesSeguimiento.includes(req.userId);
    
    res.json({
      id: user._id,
      perfil: user.perfil,
      stats: {
        publicaciones: await Post.countDocuments({ autor: user._id }),
        seguidores: user.seguidores.length,
        seguidos: user.seguidos.length
      },
      esSeguidor,
      solicitudEnviada,
      privacidad: user.privacidad
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Editar perfil
app.put('/api/users/profile', authMiddleware, async (req, res) => {
  try {
    const updates = req.body;
    const user = await User.findByIdAndUpdate(
      req.userId,
      { $set: { 'perfil': { ...updates } } },
      { new: true }
    );
    res.json(user.perfil);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Seguir/Dejar de seguir
app.post('/api/users/:id/seguir', authMiddleware, async (req, res) => {
  try {
    const targetUser = await User.findById(req.params.id);
    const currentUser = await User.findById(req.userId);
    
    if (targetUser.seguidores.includes(req.userId)) {
      // Dejar de seguir
      targetUser.seguidores.pull(req.userId);
      currentUser.seguidos.pull(req.params.id);
    } else {
      if (targetUser.privacidad.cuentaPrivada) {
        targetUser.solicitudesSeguimiento.push(req.userId);
        // Notificación de solicitud
        await Notification.create({
          destinatario: targetUser._id,
          tipo: 'solicitud',
          emisor: req.userId,
          mensaje: 'quiere seguirte'
        });
      } else {
        targetUser.seguidores.push(req.userId);
        currentUser.seguidos.push(req.params.id);
        
        // Notificación de nuevo seguidor
        await Notification.create({
          destinatario: targetUser._id,
          tipo: 'seguimiento',
          emisor: req.userId,
          mensaje: 'empezó a seguirte'
        });
      }
    }
    
    await targetUser.save();
    await currentUser.save();
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Buscar usuarios
app.get('/api/users/search/:query', authMiddleware, async (req, res) => {
  try {
    const users = await User.find({
      $or: [
        { 'perfil.username': { $regex: req.params.query, $options: 'i' } },
        { 'perfil.nombre': { $regex: req.params.query, $options: 'i' } }
      ]
    }).limit(20).select('perfil.username perfil.nombre perfil.foto');
    
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 6: RUTAS DE PUBLICACIONES
// ============================================

// Crear publicación
app.post('/api/posts', authMiddleware, upload.array('media', 10), async (req, res) => {
  try {
    const { caption, ubicacion, hashtags, etiquetas } = req.body;
    const archivos = req.files.map((file, index) => ({
      url: file.path,
      descripcion: '',
      etiquetas: etiquetas ? JSON.parse(etiquetas)[index] || [] : []
    }));
    
    const post = new Post({
      autor: req.userId,
      tipo: archivos.length > 1 ? 'carrusel' : (req.files[0].mimetype.includes('video') ? 'video' : 'foto'),
      contenido: archivos,
      caption,
      ubicacion,
      hashtags: hashtags?.split(' ') || []
    });
    
    await post.save();
    await post.populate('autor', 'perfil.username perfil.foto');
    
    // Notificar a seguidores
    const user = await User.findById(req.userId);
    for (const seguidor of user.seguidores) {
      await Notification.create({
        destinatario: seguidor,
        tipo: 'publicacion',
        emisor: req.userId,
        referencia: { tipo: 'publicacion', id: post._id },
        mensaje: 'publicó algo nuevo'
      });
    }
    
    res.status(201).json(post);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Obtener feed
app.get('/api/posts/feed', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.userId);
    const seguidosIds = user.seguidos.map(s => s.toString());
    seguidosIds.push(req.userId);
    
    const posts = await Post.find({ autor: { $in: seguidosIds } })
      .sort({ createdAt: -1 })
      .limit(20)
      .populate('autor', 'perfil.username perfil.foto')
      .populate('comentarios.autor', 'perfil.username perfil.foto');
    
    res.json(posts);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Like/Unlike
app.post('/api/posts/:id/like', authMiddleware, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    const yaDioLike = post.likes.some(l => l.usuario.toString() === req.userId);
    
    if (yaDioLike) {
      post.likes = post.likes.filter(l => l.usuario.toString() !== req.userId);
    } else {
      post.likes.push({ usuario: req.userId, timestamp: new Date() });
      // Notificación
      if (post.autor.toString() !== req.userId) {
        await Notification.create({
          destinatario: post.autor,
          tipo: 'like',
          emisor: req.userId,
          referencia: { tipo: 'publicacion', id: post._id },
          mensaje: 'le dio like a tu publicación'
        });
      }
    }
    
    await post.save();
    res.json({ likes: post.likes.length });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Comentar
app.post('/api/posts/:id/comentar', authMiddleware, async (req, res) => {
  try {
    const { texto } = req.body;
    const post = await Post.findById(req.params.id);
    
    post.comentarios.push({
      autor: req.userId,
      texto,
      createdAt: new Date()
    });
    
    await post.save();
    
    // Notificación
    if (post.autor.toString() !== req.userId) {
      await Notification.create({
        destinatario: post.autor,
        tipo: 'comentario',
        emisor: req.userId,
        referencia: { tipo: 'publicacion', id: post._id },
        mensaje: 'comentó tu publicación'
      });
    }
    
    res.json(post.comentarios[post.comentarios.length - 1]);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Guardar publicación
app.post('/api/posts/:id/guardar', authMiddleware, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    const yaGuardado = post.guardados.includes(req.userId);
    
    if (yaGuardado) {
      post.guardados.pull(req.userId);
    } else {
      post.guardados.push(req.userId);
    }
    
    await post.save();
    res.json({ guardado: !yaGuardado });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 7: RUTAS DE HISTORIAS
// ============================================

// Crear historia
app.post('/api/stories', authMiddleware, upload.single('media'), async (req, res) => {
  try {
    const { tipo, texto, colorFondo, musica, ubicacion } = req.body;
    
    const story = new Story({
      autor: req.userId,
      tipo,
      contenido: {
        url: req.file?.path || '',
        texto,
        colorFondo
      },
      filtros: {
        musica: musica ? JSON.parse(musica) : undefined,
        ubicacion
      }
    });
    
    await story.save();
    res.status(201).json(story);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Obtener historias activas
app.get('/api/stories', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.userId);
    const seguidosIds = user.seguidos.map(s => s.toString());
    
    const historias = await Story.find({
      autor: { $in: seguidosIds },
      expiraAt: { $gt: new Date() }
    })
    .populate('autor', 'perfil.username perfil.foto')
    .sort({ createdAt: -1 });
    
    // Agrupar por autor
    const agrupadas = historias.reduce((acc, h) => {
      const autorId = h.autor._id.toString();
      if (!acc[autorId]) {
        acc[autorId] = { autor: h.autor, historias: [] };
      }
      acc[autorId].historias.push(h);
      return acc;
    }, {});
    
    res.json(Object.values(agrupadas));
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Ver historia
app.post('/api/stories/:id/ver', authMiddleware, async (req, res) => {
  try {
    const story = await Story.findById(req.params.id);
    const yaVio = story.vistas.some(v => v.usuario.toString() === req.userId);
    
    if (!yaVio) {
      story.vistas.push({ usuario: req.userId, timestamp: new Date() });
      await story.save();
    }
    
    res.json({ vistas: story.vistas.length });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Destacar historia
app.post('/api/stories/:id/destacar', authMiddleware, async (req, res) => {
  try {
    const { categoria } = req.body;
    await Story.findByIdAndUpdate(req.params.id, {
      destacada: true,
      categoriaDestacada: categoria,
      expiraAt: null // No expira
    });
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 8: RUTAS DE MENSAJERÍA
// ============================================

// Obtener conversaciones
app.get('/api/conversations', authMiddleware, async (req, res) => {
  try {
    const conversations = await Conversation.find({
      participantes: req.userId
    })
    .populate('participantes', 'perfil.username perfil.foto')
    .populate('ultimoMensaje')
    .sort({ updatedAt: -1 });
    
    res.json(conversations);
  }   ultimaActividad: { type: Date, default: Date.now },
  createdAt: { type: Date, default: Date.now }
});

const postSchema = new mongoose.Schema({
  autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  tipo: { type: String, enum: ['foto', 'video', 'carrusel', 'texto'], required: true },
  contenido: [{
    url: String,
    descripcion: String,
    etiquetas: [{ 
      usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
      posicion: { x: Number, y: Number }
    }]
  }],
  caption: String,
  hashtags: [String],
  ubicacion: String,
  likes: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, timestamp: Date }],
  comentarios: [{
    autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    texto: String,
    likes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    respuestas: [{
      autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
      texto: String,
      likes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
      createdAt: { type: Date, default: Date.now }
    }],
    createdAt: { type: Date, default: Date.now }
  }],
  guardados: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  compartidos: { type: Number, default: 0 },
  ocultarLikes: { type: Boolean, default: false },
  desactivarComentarios: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
});

const storySchema = new mongoose.Schema({
  autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  tipo: { type: String, enum: ['imagen', 'video', 'texto', 'encuesta'], required: true },
  contenido: {
    url: String,
    texto: String,
    colorFondo: String,
    fuente: String,
    opciones: [{ texto: String, votos: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }] }]
  },
  filtros: {
    musica: {
      titulo: String,
      artista: String,
      tiempoInicio: Number
    },
    ubicacion: String,
    menciones: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    hashtags: [String]
  },
  vistas: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, timestamp: Date }],
  reacciones: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, tipo: String, timestamp: Date }],
  destacada: { type: Boolean, default: false },
  categoriaDestacada: String,
  expiraAt: { type: Date, default: () => new Date(Date.now() + 24 * 60 * 60 * 1000) },
  createdAt: { type: Date, default: Date.now }
});

const conversationSchema = new mongoose.Schema({
  participantes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  tipo: { type: String, enum: ['individual', 'grupo'], default: 'individual' },
  nombre: String, // para grupos
  foto: String,   // para grupos
  mensajes: [{
    autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    tipo: { type: String, enum: ['texto', 'imagen', 'video', 'audio', 'sticker', 'ubicacion', 'publicacion'] },
    contenido: String,
    publicacionRef: { type: mongoose.Schema.Types.ObjectId, ref: 'Post' },
    reacciones: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, emoji: String }],
    respondidoA: { type: mongoose.Schema.Types.ObjectId },
    eliminado: { type: Boolean, default: false },
    vistoPor: [{ usuario: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }, timestamp: Date }],
    createdAt: { type: Date, default: Date.now }
  }],
  notas: [{
    autor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    texto: String,
    musica: String,
    expiraAt: Date,
    createdAt: { type: Date, default: Date.now }
  }],
  ultimoMensaje: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  silenciado: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  fijado: { type: Boolean, default: false },
  updatedAt: { type: Date, default: Date.now }
});

const notificationSchema = new mongoose.Schema({
  destinatario: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  tipo: { type: String, enum: ['like', 'comentario', 'seguimiento', 'solicitud', 'mencion', 'mensaje', 'respuesta', 'compartido'] },
  emisor: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  referencia: {
    tipo: String, // publicacion, comentario, historia, mensaje, usuario
    id: mongoose.Schema.Types.ObjectId
  },
  mensaje: String,
  leida: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);
const Post = mongoose.model('Post', postSchema);
const Story = mongoose.model('Story', storySchema);
const Conversation = mongoose.model('Conversation', conversationSchema);
const Notification = mongoose.model('Notification', notificationSchema);

// ============================================
// SECCIÓN 3: MIDDLEWARE Y CONFIGURACIÓN
// ============================================

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: '*' } });

app.use(cors());
app.use(express.json());
app.use(express.static('public'));

// Middleware de autenticación
const authMiddleware = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'Token requerido' });
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET || 'cancha-secret-key');
    req.userId = decoded.userId;
    req.user = await User.findById(decoded.userId);
    next();
  } catch (error) {
    res.status(401).json({ error: 'Token inválido' });
  }
};

// Configuración Multer para subida de archivos
const storage = new CloudinaryStorage({
  cloudinary: cloudinary,
  params: {
    folder: 'cancha',
    allowed_formats: ['jpg', 'png', 'mp4', 'mov'],
    transformation: [{ width: 1080, height: 1080, crop: 'limit' }]
  }
});
const upload = multer({ storage });

// ============================================
// SECCIÓN 4: RUTAS DE AUTENTICACIÓN
// ============================================

// Registro
app.post('/api/auth/register', async (req, res) => {
  try {
    const { email, password, nombre, username } = req.body;
    
    const existingUser = await User.findOne({ $or: [{ email }, { 'perfil.username': username }] });
    if (existingUser) {
      return res.status(400).json({ error: 'Email o username ya existe' });
    }
    
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({
      email,
      password: hashedPassword,
      perfil: { nombre, username }
    });
    
    await user.save();
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET || 'cancha-secret-key', { expiresIn: '7d' });
    
    res.status(201).json({ token, user: { id: user._id, email, perfil: user.perfil } });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Login
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    
    if (!user || !await bcrypt.compare(password, user.password)) {
      return res.status(401).json({ error: 'Credenciales inválidas' });
    }
    
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET || 'cancha-secret-key', { expiresIn: '7d' });
    user.ultimaActividad = new Date();
    await user.save();
    
    res.json({ token, user: { id: user._id, email, perfil: user.perfil } });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 5: RUTAS DE USUARIO
// ============================================

// Obtener perfil
app.get('/api/users/:username', authMiddleware, async (req, res) => {
  try {
    const user = await User.findOne({ 'perfil.username': req.params.username })
      .populate('seguidores', 'perfil.username perfil.foto')
      .populate('seguidos', 'perfil.username perfil.foto');
    
    if (!user) return res.status(404).json({ error: 'Usuario no encontrado' });
    
    const esSeguidor = user.seguidores.some(s => s._id.toString() === req.userId);
    const solicitudEnviada = user.solicitudesSeguimiento.includes(req.userId);
    
    res.json({
      id: user._id,
      perfil: user.perfil,
      stats: {
        publicaciones: await Post.countDocuments({ autor: user._id }),
        seguidores: user.seguidores.length,
        seguidos: user.seguidos.length
      },
      esSeguidor,
      solicitudEnviada,
      privacidad: user.privacidad
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Editar perfil
app.put('/api/users/profile', authMiddleware, async (req, res) => {
  try {
    const updates = req.body;
    const user = await User.findByIdAndUpdate(
      req.userId,
      { $set: { 'perfil': { ...updates } } },
      { new: true }
    );
    res.json(user.perfil);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Seguir/Dejar de seguir
app.post('/api/users/:id/seguir', authMiddleware, async (req, res) => {
  try {
    const targetUser = await User.findById(req.params.id);
    const currentUser = await User.findById(req.userId);
    
    if (targetUser.seguidores.includes(req.userId)) {
      // Dejar de seguir
      targetUser.seguidores.pull(req.userId);
      currentUser.seguidos.pull(req.params.id);
    } else {
      if (targetUser.privacidad.cuentaPrivada) {
        targetUser.solicitudesSeguimiento.push(req.userId);
        // Notificación de solicitud
        await Notification.create({
          destinatario: targetUser._id,
          tipo: 'solicitud',
          emisor: req.userId,
          mensaje: 'quiere seguirte'
        });
      } else {
        targetUser.seguidores.push(req.userId);
        currentUser.seguidos.push(req.params.id);
        
        // Notificación de nuevo seguidor
        await Notification.create({
          destinatario: targetUser._id,
          tipo: 'seguimiento',
          emisor: req.userId,
          mensaje: 'empezó a seguirte'
        });
      }
    }
    
    await targetUser.save();
    await currentUser.save();
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Buscar usuarios
app.get('/api/users/search/:query', authMiddleware, async (req, res) => {
  try {
    const users = await User.find({
      $or: [
        { 'perfil.username': { $regex: req.params.query, $options: 'i' } },
        { 'perfil.nombre': { $regex: req.params.query, $options: 'i' } }
      ]
    }).limit(20).select('perfil.username perfil.nombre perfil.foto');
    
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 6: RUTAS DE PUBLICACIONES
// ============================================

// Crear publicación
app.post('/api/posts', authMiddleware, upload.array('media', 10), async (req, res) => {
  try {
    const { caption, ubicacion, hashtags, etiquetas } = req.body;
    const archivos = req.files.map((file, index) => ({
      url: file.path,
      descripcion: '',
      etiquetas: etiquetas ? JSON.parse(etiquetas)[index] || [] : []
    }));
    
    const post = new Post({
      autor: req.userId,
      tipo: archivos.length > 1 ? 'carrusel' : (req.files[0].mimetype.includes('video') ? 'video' : 'foto'),
      contenido: archivos,
      caption,
      ubicacion,
      hashtags: hashtags?.split(' ') || []
    });
    
    await post.save();
    await post.populate('autor', 'perfil.username perfil.foto');
    
    // Notificar a seguidores
    const user = await User.findById(req.userId);
    for (const seguidor of user.seguidores) {
      await Notification.create({
        destinatario: seguidor,
        tipo: 'publicacion',
        emisor: req.userId,
        referencia: { tipo: 'publicacion', id: post._id },
        mensaje: 'publicó algo nuevo'
      });
    }
    
    res.status(201).json(post);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Obtener feed
app.get('/api/posts/feed', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.userId);
    const seguidosIds = user.seguidos.map(s => s.toString());
    seguidosIds.push(req.userId);
    
    const posts = await Post.find({ autor: { $in: seguidosIds } })
      .sort({ createdAt: -1 })
      .limit(20)
      .populate('autor', 'perfil.username perfil.foto')
      .populate('comentarios.autor', 'perfil.username perfil.foto');
    
    res.json(posts);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Like/Unlike
app.post('/api/posts/:id/like', authMiddleware, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    const yaDioLike = post.likes.some(l => l.usuario.toString() === req.userId);
    
    if (yaDioLike) {
      post.likes = post.likes.filter(l => l.usuario.toString() !== req.userId);
    } else {
      post.likes.push({ usuario: req.userId, timestamp: new Date() });
      // Notificación
      if (post.autor.toString() !== req.userId) {
        await Notification.create({
          destinatario: post.autor,
          tipo: 'like',
          emisor: req.userId,
          referencia: { tipo: 'publicacion', id: post._id },
          mensaje: 'le dio like a tu publicación'
        });
      }
    }
    
    await post.save();
    res.json({ likes: post.likes.length });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Comentar
app.post('/api/posts/:id/comentar', authMiddleware, async (req, res) => {
  try {
    const { texto } = req.body;
    const post = await Post.findById(req.params.id);
    
    post.comentarios.push({
      autor: req.userId,
      texto,
      createdAt: new Date()
    });
    
    await post.save();
    
    // Notificación
    if (post.autor.toString() !== req.userId) {
      await Notification.create({
        destinatario: post.autor,
        tipo: 'comentario',
        emisor: req.userId,
        referencia: { tipo: 'publicacion', id: post._id },
        mensaje: 'comentó tu publicación'
      });
    }
    
    res.json(post.comentarios[post.comentarios.length - 1]);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Guardar publicación
app.post('/api/posts/:id/guardar', authMiddleware, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    const yaGuardado = post.guardados.includes(req.userId);
    
    if (yaGuardado) {
      post.guardados.pull(req.userId);
    } else {
      post.guardados.push(req.userId);
    }
    
    await post.save();
    res.json({ guardado: !yaGuardado });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 7: RUTAS DE HISTORIAS
// ============================================

// Crear historia
app.post('/api/stories', authMiddleware, upload.single('media'), async (req, res) => {
  try {
    const { tipo, texto, colorFondo, musica, ubicacion } = req.body;
    
    const story = new Story({
      autor: req.userId,
      tipo,
      contenido: {
        url: req.file?.path || '',
        texto,
        colorFondo
      },
      filtros: {
        musica: musica ? JSON.parse(musica) : undefined,
        ubicacion
      }
    });
    
    await story.save();
    res.status(201).json(story);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Obtener historias activas
app.get('/api/stories', authMiddleware, async (req, res) => {
  try {
    const user = await User.findById(req.userId);
    const seguidosIds = user.seguidos.map(s => s.toString());
    
    const historias = await Story.find({
      autor: { $in: seguidosIds },
      expiraAt: { $gt: new Date() }
    })
    .populate('autor', 'perfil.username perfil.foto')
    .sort({ createdAt: -1 });
    
    // Agrupar por autor
    const agrupadas = historias.reduce((acc, h) => {
      const autorId = h.autor._id.toString();
      if (!acc[autorId]) {
        acc[autorId] = { autor: h.autor, historias: [] };
      }
      acc[autorId].historias.push(h);
      return acc;
    }, {});
    
    res.json(Object.values(agrupadas));
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Ver historia
app.post('/api/stories/:id/ver', authMiddleware, async (req, res) => {
  try {
    const story = await Story.findById(req.params.id);
    const yaVio = story.vistas.some(v => v.usuario.toString() === req.userId);
    
    if (!yaVio) {
      story.vistas.push({ usuario: req.userId, timestamp: new Date() });
      await story.save();
    }
    
    res.json({ vistas: story.vistas.length });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Destacar historia
app.post('/api/stories/:id/destacar', authMiddleware, async (req, res) => {
  try {
    const { categoria } = req.body;
    await Story.findByIdAndUpdate(req.params.id, {
      destacada: true,
      categoriaDestacada: categoria,
      expiraAt: null // No expira
    });
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 8: RUTAS DE MENSAJERÍA
// ============================================

// Obtener conversaciones
app.get('/api/conversations', authMiddleware, async (req, res) => {
  try {
    const conversations = await Conversation.find({
      participantes: req.userId
    })
    .populate('participantes', 'perfil.username perfil.foto')
    .populate('ultimoMensaje')
    .sort({ updatedAt: -1 });
    
    res.json(conversations);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Obtener mensajes de conversación
app.get('/api/conversations/:id/messages', authMiddleware, async (req, res) => {
  try {
    const conversation = await Conversation.findOne({
      _id: req.params.id,
      participantes: req.userId
    }).populate('mensajes.autor', 'perfil.username perfil.foto');
    
    if (!conversation) return res.status(404).json({ error: 'Conversación no encontrada' });
    
    // Marcar como vistos
    conversation.mensajes.forEach(m => {
      if (!m.vistoPor.some(v => v.usuario.toString() === req.userId)) {
        m.vistoPor.push({ usuario: req.userId, timestamp: new Date() });
      }
    });
    await conversation.save();
    
    res.json(conversation.mensajes);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Enviar mensaje
app.post('/api/conversations/:id/messages', authMiddleware, async (req, res) => {
  try {
    const { tipo, contenido, publicacionRef } = req.body;
    const conversation = await Conversation.findById(req.params.id);
    
    const mensaje = {
      autor: req.userId,
      tipo,
      contenido,
      publicacionRef,
      createdAt: new Date()
    };
    
    conversation.mensajes.push(mensaje);
    conversation.ultimoMensaje = req.userId;
    conversation.updatedAt = new Date();
    await conversation.save();
    
    // Notificar vía Socket.io
    const otrosParticipantes = conversation.participantes.filter(
      p => p.toString() !== req.userId
    );
    
    otrosParticipantes.forEach(participanteId => {
      io.to(participanteId.toString()).emit('nuevoMensaje', {
        conversacion: conversation._id,
        mensaje
      });
    });
    
    res.json(mensaje);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Crear conversación
app.post('/api/conversations', authMiddleware, async (req, res) => {
  try {
    const { participanteId } = req.body;
    
    // Buscar si ya existe
    let conversation = await Conversation.findOne({
      tipo: 'individual',
      participantes: { $all: [req.userId, participanteId], $size: 2 }
    });
    
    if (!conversation) {
      conversation = new Conversation({
        participantes: [req.userId, participanteId],
        tipo: 'individual'
      });
      await conversation.save();
    }
    
    res.json(conversation);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Crear nota efímera
app.post('/api/conversations/:id/notas', authMiddleware, async (req, res) => {
  try {
    const { texto, musica } = req.body;
    const conversation = await Conversation.findById(req.params.id);
    
    conversation.notas.push({
      autor: req.userId,
      texto,
      musica,
      expiraAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
    });
    
    await conversation.save();
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 9: RUTAS DE NOTIFICACIONES
// ============================================

// Obtener notificaciones
app.get('/api/notifications', authMiddleware, async (req, res) => {
  try {
    const notificaciones = await Notification.find({ destinatario: req.userId })
      .populate('emisor', 'perfil.username perfil.foto')
      .sort({ createdAt: -1 })
      .limit(50);
    
    res.json(notificaciones);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Marcar como leídas
app.put('/api/notifications/leer', authMiddleware, async (req, res) => {
  try {
    await Notification.updateMany(
      { destinatario: req.userId, leida: false },
      { leida: true }
    );
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ============================================
// SECCIÓN 10: WEBSOCKET (SOCKET.IO)
// ============================================

io.use(async (socket, next) => {
  try {
    const token = socket.handshake.auth.token;
    const decoded = jwt.verify(token, process.env.JWT_SECRET || 'cancha-secret-key');
    socket.userId = decoded.userId;
    next();
  } catch (error) {
    next(new Error('Autenticación fallida'));
  }
});

io.on('connection', (socket) => {
  console.log('Usuario conectado:', socket.userId);
  socket.join(socket.userId);
  
  // Estado de escritura
  socket.on('escribiendo', ({ conversacionId, escribiendo }) => {
    socket.to(conversacionId).emit('usuarioEscribiendo', {
      usuarioId: socket.userId,
      escribiendo
    });
  });
  
  // Llamada de voz/video
  socket.on('llamar', ({ usuarioId, tipo }) => {
    io.to(usuarioId).emit('llamadaEntrante', {
      desde: socket.userId,
      tipo
    });
  });
  
  socket.on('disconnect', () => {
    console.log('Usuario desconectado:', socket.userId);
  });
});

// ============================================
// SECCIÓN 11: FRONTEND REACT (EMBEBIDO)
// ============================================

const htmlFrontend = `
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CANCHA - Red Social de la Vida</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/socket.io-client@4/dist/socket.io.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
        body { background: #000; color: #fff; }
        .app { max-width: 480px; margin: 0 auto; min-height: 100vh; position: relative; }
        
        /* Login/Register */
        .auth-container { display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 100vh; padding: 20px; }
        .auth-logo { font-size: 48px; font-weight: bold; margin-bottom: 40px; background: linear-gradient(45deg, #f09433, #e6683c, #dc2743, #cc2366, #bc1888); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .auth-form { width: 100%; max-width: 350px; }
        .auth-input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #333; border-radius: 8px; background: #121212; color: #fff; font-size: 14px; }
        .auth-button { width: 100%; padding: 12px; margin: 16px 0; background: #0095f6; border: none; border-radius: 8px; color: #fff; font-weight: 600; cursor: pointer; }
        .auth-button:disabled { opacity: 0.5; }
        .auth-link { color: #0095f6; text-decoration: none; cursor: pointer; }
        
        /* Header */
        .header { position: fixed; top: 0; width: 100%; max-width: 480px; background: #000; border-bottom: 1px solid #262626; padding: 12px 16px; display: flex; justify-content: space-between; align-items: center; z-index: 100; }
        .header-logo { font-size: 24px; font-weight: bold; }
        .header-icons { display: flex; gap: 20px; }
        .header-icon { width: 24px; height: 24px; cursor: pointer; }
        
        /* Stories */
        .stories-container { display: flex; gap: 12px; padding: 80px 16px 12px; overflow-x: auto; border-bottom: 1px solid #262626; }
        .story-item { display: flex; flex-direction: column; align-items: center; gap: 4px; cursor: pointer; flex-shrink: 0; }
        .story-ring { width: 66px; height: 66px; border-radius: 50%; padding: 3px; background: linear-gradient(45deg, #f09433, #e6683c, #dc2743, #cc2366, #bc1888); }
        .story-ring-visto { background: #333; }
        .story-img { width: 60px; height: 60px; border-radius: 50%; border: 3px solid #000; object-fit: cover; }
        .story-username { font-size: 12px; color: #fff; max-width: 70px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        .story-add { position: relative; }
        .story-add-icon { position: absolute; bottom: 0; right: 0; background: #0095f6; border-radius: 50%; width: 20px; height: 20px; display: flex; align-items: center; justify-content: center; font-size: 16px; border: 3px solid #000; }
        
        /* Feed */
        .feed { padding-bottom: 60px; }
        .post { border-bottom: 1px solid #262626; margin-bottom: 12px; }
        .post-header { display: flex; align-items: center; padding: 12px 16px; gap: 12px; }
        .post-avatar { width: 32px; height: 32px; border-radius: 50%; object-fit: cover; }
        .post-username { font-weight: 600; font-size: 14px; }
        .post-location { font-size: 12px; color: #8e8e8e; }
        .post-more { margin-left: auto; cursor: pointer; }
        .post-media { width: 100%; aspect-ratio: 1; object-fit: cover; }
        .post-actions { display: flex; padding: 12px 16px; gap: 16px; }
        .post-action { cursor: pointer; font-size: 24px; }
        .post-action-save { margin-left: auto; }
        .post-likes { padding: 0 16px 8px; font-weight: 600; font-size: 14px; }
        .post-caption { padding: 0 16px 12px; font-size: 14px; }
        .post-caption-username { font-weight: 600; margin-right: 4px; }
        .post-comments { padding: 0 16px 12px; color: #8e8e8e; font-size: 14px; cursor: pointer; }
        .post-time { padding: 0 16px 12px; color: #8e8e8e; font-size: 10px; text-transform: uppercase; }
        
        /* Bottom Nav */
        .bottom-nav { position: fixed; bottom: 0; width: 100%; max-width: 480px; background: #000; border-top: 1px solid #262626; display: flex; justify-content: space-around; padding: 8px 0; z-index: 100; }
        .nav-item { font-size: 24px; cursor: pointer; padding: 8px 16px; }
        .nav-item-active { color: #fff; }
        
        /* Profile */
        .profile-header { padding: 80px 16px 16px; }
        .profile-info { display: flex; align-items: center; gap: 20px; margin-bottom: 16px; }
        .profile-avatar { width: 80px; height: 80px; border-radius: 50%; object-fit: cover; }
        .profile-stats { display: flex; gap: 24px; flex: 1; justify-content: center; }
        .profile-stat { text-align: center; }
        .profile-stat-number { font-weight: 600; font-size: 16px; }
        .profile-stat-label { font-size: 12px; color: #8e8e8e; }
        .profile-name { font-weight: 600; margin-bottom: 4px; }
        .profile-bio { font-size: 14px; margin-bottom: 12px; }
        .profile-buttons { display: flex; gap: 8px; margin-bottom: 16px; }
        .profile-button { flex: 1; padding: 8px; background: #262626; border: none; border-radius: 8px; color: #fff; font-weight: 600; cursor: pointer; }
        .profile-button-primary { background: #0095f6; }
        
        /* Stories Highlights */
        .highlights { display: flex; gap: 16px; padding: 16px; overflow-x: auto; border-bottom: 1px solid #262626; }
        .highlight { display: flex; flex-direction: column; align-items: center; gap: 4px; cursor: pointer; }
        .highlight-img { width: 60px; height: 60px; border-radius: 50%; border: 1px solid #333; padding: 2px; object-fit: cover; }
        .highlight-name { font-size: 12px; max-width: 70px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        
        /* Grid */
        .profile-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 2px; }
        .grid-item { aspect-ratio: 1; background: #262626; position: relative; cursor: pointer; }
        .grid-item img { width: 100%; height: 100%; object-fit: cover; }
        .grid-item-multi::after { content: '▣'; position: absolute; top: 8px; right: 8px; color: #fff; font-size: 16px; text-shadow: 0 1px 3px rgba(0,0,0,0.5); }
        
        /* Create Post Modal */
        .modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.9); z-index: 1000; display: flex; align-items: center; justify-content: center; }
        .modal-content { background: #121212; width: 90%; max-width: 400px; border-radius: 12px; overflow: hidden; }
        .modal-header { display: flex; justify-content: space-between; align-items: center; padding: 12px 16px; border-bottom: 1px solid #262626; }
        .modal-title { font-weight: 600; }
        .modal-close { background: none; border: none; color: #fff; font-size: 20px; cursor: pointer; }
        .modal-body { padding: 16px; }
        .file-input { display: none; }
        .file-label { display: block; padding: 40px; text-align: center; border: 2px dashed #333; border-radius: 8px; cursor: pointer; color: #8e8e8e; }
        .caption-input { width: 100%; padding: 12px; margin-top: 12px; background: #000; border: 1px solid #333; border-radius: 8px; color: #fff; resize: none; }
        
        /* Messages */
        .messages-container { padding: 80px 16px 80px; height: 100vh; overflow-y: auto; }
        .message-item { display: flex; align-items: center; gap: 12px; padding: 12px 0; border-bottom: 1px solid #262626; cursor: pointer; }
        .message-avatar { width: 56px; height: 56px; border-radius: 50%; object-fit: cover; }
        .message-content { flex: 1; }
        .message-username { font-weight: 600; margin-bottom: 4px; display: flex; align-items: center; gap: 4px; }
        .message-preview { color: #8e8e8e; font-size: 14px; }
        .message-time { color: #8e8e8e; font-size: 12px; }
        .message-unread { width: 8px; height: 8px; background: #0095f6; border-radius: 50%; margin-left: auto; }
        
        /* Chat */
        .chat-container { height: 100vh; display: flex; flex-direction: column; }
        .chat-header { display: flex; align-items: center; padding: 12px 16px; border-bottom: 1px solid #262626; gap: 12px; }
        .chat-back { font-size: 24px; cursor: pointer; }
        .chat-messages { flex: 1; overflow-y: auto; padding: 16px; display: flex; flex-direction: column; gap: 8px; }
        .message-bubble { max-width: 70%; padding: 12px 16px; border-radius: 20px; font-size: 14px; }
        .message-sent { align-self: flex-end; background: #0095f6; }
        .message-received { align-self: flex-start; background: #262626; }
        .chat-input-container { display: flex; align-items: center; padding: 12px 16px; gap: 12px; border-top: 1px solid #262626; }
        .chat-input { flex: 1; padding: 12px 16px; background: #262626; border: none; border-radius: 20px; color: #fff; outline: none; }
        .chat-send { background: none; border: none; color: #0095f6; font-size: 20px; cursor: pointer; }
        
        /* Notifications */
        .notifications-container { padding: 80px 16px 80px; }
        .notification-item { display: flex; align-items: center; gap: 12px; padding: 12px 0; border-bottom: 1px solid #262626; }
        .notification-avatar { width: 44px; height: 44px; border-radius: 50%; object-fit: cover; }
        .notification-content { flex: 1; font-size: 14px; }
        .notification-username { font-weight: 600; }
        .notification-time { color: #8e8e8e; font-size: 12px; margin-top: 4px; }
        .notification-post { width: 44px; height: 44px; object-fit: cover; border-radius: 4px; }
        .notification-follow { padding: 6px 16px; background: #0095f6; border: none; border-radius: 8px; color: #fff; font-weight: 600; cursor: pointer; }
        
        /* Search */
        .search-container { padding: 80px 16px 80px; }
        .search-input { width: 100%; padding: 12px 16px; background: #262626; border: none; border-radius: 8px; color: #fff; margin-bottom: 16px; }
        .search-results { display: flex; flex-direction: column; gap: 12px; }
        .search-item { display: flex; align-items: center; gap: 12px; padding: 8px 0; cursor: pointer; }
        .search-avatar { width: 44px; height: 44px; border-radius: 50%; object-fit: cover; }
        .search-info { flex: 1; }
        .search-username { font-weight: 600; }
        .search-name { color: #8e8e8e; font-size: 14px; }
        
        /* Loading */
        .loading { display: flex; justify-content: center; align-items: center; height: 100vh; }
        .spinner { width: 40px; height: 40px; border: 4px solid #333; border-top-color: #0095f6; border-radius: 50%; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
        
        /* Story Viewer */
        .story-viewer { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: #000; z-index: 2000; display: flex; flex-direction: column; }
        .story-progress { display: flex; gap: 4px; padding: 12px; }
        .story-progress-bar { flex: 1; height: 2px; background: rgba(255,255,255,0.3); border-radius: 2px; overflow: hidden; }
        .story-progress-fill { height: 100%; background: #fff; width: 0%; transition: width 0.1s linear; }
        .story-header { display: flex; align-items: center; padding: 12px; gap: 12px; }
        .story-close { margin-left: auto; font-size: 24px; cursor: pointer; }
        .story-content { flex: 1; display: flex; align-items: center; justify-content: center; }
        .story-content img, .story-content video { max-width: 100%; max-height: 100%; object-fit: contain; }
        .story-footer { padding: 20px; display: flex; gap: 12px; }
        .story-input { flex: 1; padding: 12px 16px; background: rgba(255,255,255,0.1); border: none; border-radius: 20px; color: #fff; }
    </style>
</head>
<body>
    <div id="root"></div>
    
    <script type="text/babel">
        const { useState, useEffect, useRef, createContext, useContext } = React;
        
        // Contexto de autenticación
        const AuthContext = createContext();
        
        const AuthProvider = ({ children }) => {
            const [user, setUser] = useState(null);
            const [token, setToken] = useState(localStorage.getItem('cancha_token'));
            const [loading, setLoading] = useState(true);
            
            useEffect(() => {
                if (token) {
                    fetchUser();
                } else {
                    setLoading(false);
                }
            }, [token]);
            
            const fetchUser = async () => {
                try {
                    const res = await fetch('/api/users/me', {
                        headers: { 'Authorization': 'Bearer ' + token }
                    });
                    if (res.ok) {
                        const data = await res.json();
                        setUser(data);
                    } else {
                        logout();
                    }
                } catch (error) {
                    logout();
                }
                setLoading(false);
            };
            
            const login = (newToken, userData) => {
                localStorage.setItem('cancha_token', newToken);
                setToken(newToken);
                setUser(userData);
            };
            
            const logout = () => {
                localStorage.removeItem('cancha_token');
                setToken(null);
                setUser(null);
            };
            
            return (
                <AuthContext.Provider value={{ user, token, login, logout, loading }}>
                    {children}
                </AuthContext.Provider>
            );
        };
        
        const useAuth = () => useContext(AuthContext);
        
        // API helper
const api = {
    get: (url, token) => fetch(url, { headers: { 'Authorization': 'Bearer ' + token } }).then(r => r.json()),
    post: (url, data, token) => fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer ' + token },
        body: JSON.stringify(data)
    }).then(r => r.json()),
    upload: (url, formData, token) => fetch(url, {
        method: 'POST',
        headers: { 'Authorization': 'Bearer ' + token },
        body: formData
    }).then(r => r.json())
};

// Componente Login
const Login = ({ onRegister }) => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [loading, setLoading] = useState(false);
    const { login } = useAuth();
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);
        try {
            const res = await fetch('/api/auth/login', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ email, password })
            });
            const data = await res.json();
            if (data.token) {
                login(data.token, data.user);
            } else {
                alert(data.error);
            }
        } catch (error) {
            alert('Error de conexión');
        }
        setLoading(false);
    };
    
    return (
        <div className="auth-container">
            <div className="auth-logo">CANCHA</div>
            <form className="auth-form" onSubmit={handleSubmit}>
                <input 
                    type="email" 
                    className="auth-input" 
                    placeholder="Correo electrónico" 
                    value={email} 
                    onChange={e => setEmail(e.target.value)} 
                    required 
                />
                <input 
                    type="password" 
                    className="auth-input" 
                    placeholder="Contraseña" 
                    value={password} 
                    onChange={e => setPassword(e.target.value)} 
                    required 
                />
                <button type="submit" className="auth-button" disabled={loading}>
                    {loading ? 'Entrando...' : 'Entrar'}
                </button>
            </form>
            <p style={{marginTop: 20}}>
                ¿No tienes cuenta? <span className="auth-link" onClick={onRegister}>Regístrate</span>
            </p>
        </div>
    );
};

// Componente Register
const Register = ({ onLogin }) => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [nombre, setNombre] = useState('');
    const [username, setUsername] = useState('');
    const [loading, setLoading] = useState(false);
    const { login } = useAuth();
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);
        try {
            const res = await fetch('/api/auth/register', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ email, password, nombre, username })
            });
            const data = await res.json();
            if (data.token) {
                login(data.token, data.user);
            } else {
                alert(data.error);
            }
        } catch (error) {
            alert('Error de conexión');
        }
        setLoading(false);
    };
    
    return (
        <div className="auth-container">
            <div className="auth-logo">CANCHA</div>
            <form className="auth-form" onSubmit={handleSubmit}>
                <input 
                    type="email" 
                    className="auth-input" 
                    placeholder="Correo electrónico" 
                    value={email} 
                    onChange={e => setEmail(e.target.value)} 
                    required 
                />
                <input 
                    type="text" 
                    className="auth-input" 
                    placeholder="Nombre completo" 
                    value={nombre} 
                    onChange={e => setNombre(e.target.value)} 
                    required 
                />
                <input 
                    type="text" 
                    className="auth-input" 
                    placeholder="Nombre de usuario" 
                    value={username} 
                    onChange={e => setUsername(e.target.value)} 
                    required 
                />
                <input 
                    type="password" 
                    className="auth-input" 
                    placeholder="Contraseña" 
                    value={password} 
                    onChange={e => setPassword(e.target.value)} 
                    required 
                />
                <button type="submit" className="auth-button" disabled={loading}>
                    {loading ? 'Creando cuenta...' : 'Registrarse'}
                </button>
            </form>
            <p style={{marginTop: 20}}>
                ¿Ya tienes cuenta? <span className="auth-link" onClick={onLogin}>Inicia sesión</span>
            </p>
        </div>
    );
};

// Componente Header
const Header = ({ onCreate }) => {
    const { logout } = useAuth();
    return (
        <div className="header">
            <div className="header-logo">CANCHA</div>
            <div className="header-icons">
                <span className="header-icon" onClick={onCreate}>+</span>
                <span className="header-icon" onClick={logout}>⚙</span>
            </div>
        </div>
    );
};

// Componente Stories
const Stories = ({ onViewStory }) => {
    const [stories, setStories] = useState([]);
    const { token, user } = useAuth();
    
    useEffect(() => {
        loadStories();
    }, []);
    
    const loadStories = async () => {
        try {
            const data = await api.get('/api/stories', token);
            setStories(data);
        } catch (error) {
            console.error(error);
        }
    };
    
    return (
        <div className="stories-container">
            <div className="story-item story-add">
                <div className="story-ring">
                    <img src={user?.perfil?.foto} className="story-img" alt="Tú" />
                </div>
                <div className="story-add-icon">+</div>
                <span className="story-username">Tu historia</span>
            </div>
            {stories.map(grupo => (
                <div key={grupo.autor._id} className="story-item" onClick={() => onViewStory(grupo)}>
                    <div className="story-ring">
                        <img src={grupo.autor.perfil.foto} className="story-img" alt={grupo.autor.perfil.username} />
                    </div>
                    <span className="story-username">{grupo.autor.perfil.username}</span>
                </div>
            ))}
        </div>
    );
};

// Componente StoryViewer
const StoryViewer = ({ grupo, onClose }) => {
    const [currentIndex, setCurrentIndex] = useState(0);
    const [progress, setProgress] = useState(0);
    const { token } = useAuth();
    
    useEffect(() => {
        const interval = setInterval(() => {
            setProgress(p => {
                if (p >= 100) {
                    if (currentIndex < grupo.historias.length - 1) {
                        setCurrentIndex(i => i + 1);
                        return 0;
                    } else {
                        onClose();
                        return 100;
                    }
                }
                return p + 2;
            });
        }, 100);
        
        // Marcar como vista
        api.post('/api/stories/' + grupo.historias[currentIndex]._id + '/ver', {}, token);
        
        return () => clearInterval(interval);
    }, [currentIndex]);
    
    const historia = grupo.historias[currentIndex];
    
    return (
        <div className="story-viewer" onClick={onClose}>
            <div className="story-progress">
                {grupo.historias.map((_, i) => (
                    <div key={i} className="story-progress-bar">
                        <div 
                            className="story-progress-fill" 
                            style={{width: i < currentIndex ? '100%' : i === currentIndex ? progress + '%' : '0%'}}
                        />
                    </div>
                ))}
            </div>
            <div className="story-header">
                <img src={grupo.autor.perfil.foto} className="story-avatar" style={{width: 32, height: 32}} />
                <span className="story-username">{grupo.autor.perfil.username}</span>
                <span className="story-close">✕</span>
            </div>
            <div className="story-content">
                {historia.tipo === 'video' ? (
                    <video src={historia.contenido.url} autoPlay muted />
                ) : (
                    <img src={historia.contenido.url} alt="" />
                )}
            </div>
            <div className="story-footer">
                <input type="text" className="story-input" placeholder="Enviar mensaje..." />
            </div>
        </div>
    );
};

// Componente Post
const Post = ({ post, onLike, onComment }) => {
    const [liked, setLiked] = useState(post.likes.some(l => l.usuario === post.autor._id));
    const [saved, setSaved] = useState(false);
    const [showComments, setShowComments] = useState(false);
    const [comment, setComment] = useState('');
    const { token, user } = useAuth();
    
    const handleLike = async () => {
        await api.post('/api/posts/' + post._id + '/like', {}, token);
        setLiked(!liked);
        onLike();
    };
    
    const handleSave = async () => {
        await api.post('/api/posts/' + post._id + '/guardar', {}, token);
        setSaved(!saved);
    };
    
    const handleComment = async (e) => {
        e.preventDefault();
        await api.post('/api/posts/' + post._id + '/comentar', { texto: comment }, token);
        setComment('');
        onComment();
    };
    
    return (
        <div className="post">
            <div className="post-header">
                <img src={post.autor.perfil.foto} className="post-avatar" />
                <div>
                    <div className="post-username">{post.autor.perfil.username}</div>
                    {post.ubicacion && <div className="post-location">{post.ubicacion}</div>}
                </div>
                <span className="post-more">⋯</span>
            </div>
            
            <img src={post.contenido[0]?.url} className="post-media" />
            
            <div className="post-actions">
                <span className="post-action" onClick={handleLike} style={{color: liked ? '#ed4956' : '#fff'}}>
                    {liked ? '❤' : '🤍'}
                </span>
                <span className="post-action">💬</span>
                <span className="post-action">↗</span>
                <span className="post-action post-action-save" onClick={handleSave}>
                    {saved ? '🔖' : '📑'}
                </span>
            </div>
            
            <div className="post-likes">{post.likes.length} Me gusta</div>
            
            <div className="post-caption">
                <span className="post-caption-username">{post.autor.perfil.username}</span>
                {post.caption}
            </div>
            
            <div className="post-comments" onClick={() => setShowComments(!showComments)}>
                Ver los {post.comentarios.length} comentarios
            </div>
            
            {showComments && (
                <div style={{padding: '0 16px 12px'}}>
                    {post.comentarios.map(c => (
                        <div key={c._id} style={{marginBottom: 8}}>
                            <span className="post-caption-username">{c.autor.perfil.username}</span>
                            {c.texto}
                        </div>
                    ))}
                    <form onSubmit={handleComment}>
                        <input 
                            type="text" 
                            placeholder="Añade un comentario..." 
                            value={comment}
                            onChange={e => setComment(e.target.value)}
                            style={{width: '100%', padding: 8, background: '#000', border: 'none', color: '#fff'}}
                        />
                    </form>
                </div>
            )}
            
            <div className="post-time">
                {new Date(post.createdAt).toLocaleDateString()}
            </div>
        </div>
    );
};

// Componente Feed
const Feed = () => {
    const [posts, setPosts] = useState([]);
    const [viewingStory, setViewingStory] = useState(null);
    const { token } = useAuth();
    
    useEffect(() => {
        loadPosts();
    }, []);
    
    const loadPosts = async () => {
        const data = await api.get('/api/posts/feed', token);
        setPosts(data);
    };
    
    return (
        <div className="feed">
            <Stories onViewStory={setViewingStory} />
            {posts.map(post => (
                <Post key={post._id} post={post} onLike={loadPosts} onComment={loadPosts} />
            ))}
            {viewingStory && <StoryViewer grupo={viewingStory} onClose={() => setViewingStory(null)} />}
        </div>
    );
};

// Componente Profile
const Profile = ({ username }) => {
    const [profile, setProfile] = useState(null);
    const [posts, setPosts] = useState([]);
    const [activeTab, setActiveTab] = useState('grid');
    const { token, user } = useAuth();
    
    useEffect(() => {
        loadProfile();
    }, [username]);
    
    const loadProfile = async () => {
        const data = await api.get('/api/users/' + (username || user.perfil.username), token);
        setProfile(data);
        // Cargar posts del usuario
        const userPosts = await api.get('/api/users/' + data.id + '/posts', token);
        setPosts(userPosts);
    };
    
    const handleFollow = async () => {
        await api.post('/api/users/' + profile.id + '/seguir', {}, token);
        loadProfile();
    };
    
    if (!profile) return <div className="loading"><div className="spinner"></div></div>;
    
    const esMiPerfil = profile.id === user.id;
    
    return (
        <div>
            <div className="profile-header">
                <div className="profile-info">
                    <img src={profile.perfil.foto} className="profile-avatar" />
                    <div className="profile-stats">
                        <div className="profile-stat">
                            <div className="profile-stat-number">{profile.stats.publicaciones}</div>
                            <div className="profile-stat-label">publicaciones</div>
                        </div>
                        <div className="profile-stat">
                            <div className="profile-stat-number">{profile.stats.seguidores}</div>
                            <div className="profile-stat-label">seguidores</div>
                        </div>
                        <div className="profile-stat">
                            <div className="profile-stat-number">{profile.stats.seguidos}</div>
                            <div className="profile-stat-label">seguidos</div>
                        </div>
                    </div>
                </div>
                
                <div className="profile-name">{profile.perfil.nombre}</div>
                <div className="profile-bio">{profile.perfil.biografia}</div>
                
                <div className="profile-buttons">
                    {esMiPerfil ? (
                        <button className="profile-button">Editar perfil</button>
                    ) : (
                        <button 
                            className={'profile-button ' + (profile.esSeguidor ? '' : 'profile-button-primary')}
                            onClick={handleFollow}
                        >
                            {profile.esSeguidor ? 'Dejar de seguir' : profile.solicitudEnviada ? 'Solicitado' : 'Seguir'}
                        </button>
                    )}
                    <button className="profile-button">Compartir perfil</button>
                </div>
            </div>
            
            <div className="highlights">
                <div className="highlight">
                    <div className="highlight-img" style={{display: 'flex', alignItems: 'center', justifyContent: 'center', fontSize: 24}}>+</div>
                    <span className="highlight-name">Nuevo</span>
                </div>
                <div className="highlight">
                    <img src={profile.perfil.foto} className="highlight-img" />
                    <span className="highlight-name">Destacados</span>
                </div>
            </div>
            
            <div style={{display: 'flex', justifyContent: 'space-around', borderTop: '1px solid #262626', padding: '12px 0'}}>
                <span onClick={() => setActiveTab('grid')} style={{fontSize: 24, opacity: activeTab === 'grid' ? 1 : 0.5}}>⊞</span>
                <span onClick={() => setActiveTab('reels')} style={{fontSize: 24, opacity: activeTab === 'reels' ? 1 : 0.5}}>▶</span>
                <span onClick={() => setActiveTab('tags')} style={{fontSize: 24, opacity: activeTab === 'tags' ? 1 : 0.5}}>@</span>
            </div>
            
            <div className="profile-grid">
                {posts.map(post => (
                    <div key={post._id} className={'grid-item ' + (post.contenido.length > 1 ? 'grid-item-multi' : '')}>
                        <img src={post.contenido[0]?.url} />
                    </div>
                ))}
            </div>
        </div>
    );
};

// Componente Messages
const Messages = ({ onChat }) => {
    const [conversations, setConversations] = useState([]);
    const { token } = useAuth();
    
    useEffect(() => {
        loadConversations();
    }, []);
    
    const loadConversations = async () => {
        const data = await api.get('/api/conversations', token);
        setConversations(data);
    };
    
    return (
        <div className="messages-container">
            <h2 style={{marginBottom: 16}}>Mensajes</h2>
            {conversations.map(conv => {
                const otro = conv.participantes.find(p => p._id !== conv.ultimoMensaje);
                const ultimoMsg = conv.mensajes[conv.mensajes.length - 1];
                return (
                    <div key={conv._id} className="message-item" onClick={() => onChat(conv)}>
                        <img src={otro?.perfil?.foto} className="message-avatar" />
                        <div className="message-content">
                            <div className="message-username">{otro?.perfil?.username}</div>
                            <div className="message-preview">
                                {ultimoMsg?.tipo === 'imagen' ? '📷 Foto' : ultimoMsg?.contenido}
                            </div>
                        </div>
                        <span className="message-time">
                            {ultimoMsg && new Date(ultimoMsg.createdAt).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}
                        </span>
                        {!ultimoMsg?.vistoPor?.some(v => v.usuario === otro?._id) && <div className="message-unread"></div>}
                    </div>
                );
            })}
        </div>
    );
};

// Componente Chat
const Chat = ({ conversation, onBack }) => {
    const [messages, setMessages] = useState([]);
    const [newMessage, setNewMessage] = useState('');
    const { token, user } = useAuth();
    const messagesEndRef = useRef(null);
    
    useEffect(() => {
        loadMessages();
        const socket = io({
            auth: { token }
        });
        
        socket.on('nuevoMensaje', (data) => {
            if (data.conversacion === conversation._id) {
                setMessages(prev => [...prev, data.mensaje]);
            }
        });
        
        return () => socket.disconnect();
    }, []);
    
    useEffect(() => {
        messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
    }, [messages]);
    
    const loadMessages = async () => {
        const data = await api.get('/api/conversations/' + conversation._id + '/messages', token);
        setMessages(data);
    };
    
    const handleSend = async (e) => {
        e.preventDefault();
        if (!newMessage.trim()) return;
        
        await api.post('/api/conversations/' + conversation._id + '/messages', {
            tipo: 'texto',
            contenido: newMessage
        }, token);
        
        setNewMessage('');
        loadMessages();
    };
    
    const otro = conversation.participantes.find(p => p._id !== user.id);
    
    return (
        <div className="chat-container">
            <div className="chat-header">
                <span className="chat-back" onClick={onBack}>←</span>
                <img src={otro?.perfil?.foto} className="post-avatar" />
                <span className="post-username">{otro?.perfil?.username}</span>
            </div>
            
            <div className="chat-messages">
                {messages.map((msg, i) => (
                    <div 
                        key={i} 
                        className={'message-bubble ' + (msg.autor === user.id ? 'message-sent' : 'message-received')}
                    >
                        {msg.contenido}
                    </div>
                ))}
                <div ref={messagesEndRef} />
            </div>
            
            <form className="chat-input-container" onSubmit={handleSend}>
                <span style={{fontSize: 24}}>😊</span>
                <input 
                    type="text" 
                    className="chat-input" 
                    placeholder="Mensaje..." 
                    value={newMessage}
                    onChange={e => setNewMessage(e.target.value)}
                />
                <button type="submit" className="chat-send">➤</button>
            </form>
        </div>
    );
};

// Componente Notifications
const Notifications = () => {
    const [notifications, setNotifications] = useState([]);
    const { token } = useAuth();
    
    useEffect(() => {
        loadNotifications();
        api.put('/api/notifications/leer', {}, token);
    }, []);
    
    const loadNotifications = async () => {
        const data = await api.get('/api/notifications', token);
        setNotifications(data);
    };
    
    return (
        <div className="notifications-container">
            <h2 style={{marginBottom: 16}}>Notificaciones</h2>
            {notifications.map(notif => (
                <div key={notif._id} className="notification-item">
                    <img src={notif.emisor?.perfil?.foto} className="notification-avatar" />
                    <div className="notification-content">
                        <span className="notification-username">{notif.emisor?.perfil?.username}</span>
                        {' '}{notif.mensaje}
                        <div className="notification-time">
                            {new Date(notif.createdAt).toLocaleDateString()}
                        </div>
                    </div>
                    {notif.tipo === 'seguimiento' && (
                        <button className="notification-follow">Seguir</button>
                    )}
                </div>
            ))}
        </div>
    );
};

// Componente Search
const Search = ({ onUserClick }) => {
    const [query, setQuery] = useState('');
    const [results, setResults] = useState([]);
    const { token } = useAuth();
    
    useEffect(() => {
        if (query.length > 2) {
            searchUsers();
        } else {
            setResults([]);
        }
    }, [query]);
    
    const searchUsers = async () => {
        const data = await api.get('/api/users/search/' + query, token);
        setResults(data);
    };
    
    return (
        <div className="search-container">
            <input 
                type="text" 
                className="search-input" 
                placeholder="Buscar" 
                value={query}
                onChange={e => setQuery(e.target.value)}
            />
            <div className="search-results">
                {results.map(user => (
                    <div key={user._id} className="search-item" onClick={() => onUserClick(user.perfil.username)}>
                        <img src={user.perfil.foto} className="search-avatar" />
                        <div className="search-info">
                            <div className="search-username">{user.perfil.username}</div>
                            <div className="search-name">{user.perfil.nombre}</div>
                        </div>
                    </div>
                ))}
            </div>
        </div>
    );
};

// Componente CreatePost
const CreatePost = ({ onClose, onSuccess }) => {
    const [file, setFile] = useState(null);
    const [caption, setCaption] = useState('');
    const [preview, setPreview] = useState(null);
    const { token } = useAuth();
    
    const handleFileChange = (e) => {
        const selected = e.target.files[0];
        setFile(selected);
        setPreview(URL.createObjectURL(selected));
    };
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        if (!file) return;
        
        const formData = new FormData();
        formData.append('media', file);
        formData.append('caption', caption);
        
        await api.upload('/api/posts', formData, token);
        onSuccess();
        onClose();
    };
    
    return (
        <div className="modal-overlay">
            <div className="modal-content">
                <div className="modal-header">
                    <span className="modal-title">Crear publicación</span>
                    <button className="modal-close" onClick={onClose}>✕</button>
                </div>
                <div className="modal-body">
                    {!preview ? (
                        <label className="file-label">
                            <input type="file" className="file-input" accept="image/*,video/*" onChange={handleFileChange} />
                            Selecciona una foto o video
                        </label>
                    ) : (
                        <div>
                            <img src={preview} style={{width: '100%', borderRadius: 8, marginBottom: 12}} />
                            <textarea 
                                className="caption-input" 
                                rows="3" 
                                placeholder="Escribe una descripción..."
                                value={caption}
                                onChange={e => setCaption(e.target.value)}
                            />
                            <button className="auth-button" onClick={handleSubmit}>Compartir</button>
                        </div>
                    )}
                </div>
            </div>
        </div>
    );
};

// Componente principal App
const App = () => {
    const [view, setView] = useState('feed');
    const [showCreate, setShowCreate] = useState(false);
    const [selectedUser, setSelectedUser] = useState(null);
    const [activeConversation, setActiveConversation] = useState(null);
    const { user, loading } = useAuth();
    
    if (loading) return <div className="loading"><div className="spinner"></div></div>;
    
    if (!user) {
        return view === 'register' 
            ? <Register onLogin={() => setView('login')} /> 
            : <Login onRegister={() => setView('register')} />;
    }
    
    const renderContent = () => {
        if (activeConversation) {
            return <Chat conversation={activeConversation} onBack={() => setActiveConversation(null)} />;
        }
        
        switch(view) {
            case 'feed':
                return <Feed />;
            case 'search':
                return <Search onUserClick={(username) => { setSelectedUser(username); setView('profile'); }} />;
            case 'messages':
                return <Messages onChat={setActiveConversation} />;
            case 'notifications':
                return <Notifications />;
            case 'profile':
                return <Profile username={selectedUser} />;
            default:
                return <Feed />;
        }
    };
    
    return (
        <div className="app">
            {!activeConversation && <Header onCreate={() => setShowCreate(true)} />}
            {renderContent()}
            
            {!activeConversation && (
                <div className="bottom-nav">
                    <span className={'nav-item ' + (view === 'feed' ? 'nav-item-active' : '')} onClick={() => setView('feed')}>🏠</span>
                    <span className={'nav-item ' + (view === 'search' ? 'nav-item-active' : '')} onClick={() => setView('search')}>🔍</span>
                    <span className="nav-item" onClick={() => setShowCreate(true)}>➕</span>
                    <span className={'nav-item ' + (view === 'messages' ? 'nav-item-active' : '')} onClick={() => setView('messages')}>✉</span>
                    <span className={'nav-item ' + (view === 'profile' ? 'nav-item-active' : '')} onClick={() => { setSelectedUser(null); setView('profile'); }}>👤</span>
                </div>
            )}
            
            {showCreate && <CreatePost onClose={() => setShowCreate(false)} onSuccess={() => setView('feed')} />}
        </div>
    );
};

// Renderizar aplicación
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
    <AuthProvider>
        <App />
    </AuthProvider>
);

