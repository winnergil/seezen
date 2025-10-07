# seezen
Contact &amp; well‑being manager – Hakuna Matata ✌️ 
# Remplacez  par winnergil
git clone https://github.com//seezen.git
cd seezen
# Dossier mobile (Flutter)
mkdir -p mobile/lib/screens
mkdir -p mobile/lib/providers
mkdir -p mobile/lib/services
mkdir -p mobile/lib/models
mkdir -p mobile/lib/l10n

Dossier backend (NestJS)

mkdir -p backend/src
mkdir -p backend/prisma
Dossier infra (Docker, CI/CD, etc.)

mkdir -p infra
mobile/lib/main.dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:provider/provider.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

import 'screens/auth_screen.dart';
import 'screens/home_screen.dart';
import 'screens/contacts_screen.dart';
import 'screens/wellbeing_screen.dart';
import 'screens/settings_screen.dart';
import 'providers/auth_provider.dart';
import 'providers/contacts_provider.dart';
import 'providers/wellbeing_provider.dart';
import 'providers/plan_provider.dart';
import 'services/notification_service.dart';
import 'theme.dart';
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  await NotificationService().init();
  runApp(const SeeZenApp());
}
class SeeZenApp extends StatelessWidget {
  const SeeZenApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: () => AuthProvider()),
        ChangeNotifierProvider(create: () => ContactsProvider()),
        ChangeNotifierProvider(create: () => WellbeingProvider()),
        ChangeNotifierProvider(create: () => PlanProvider()),
      ],
      child: Consumer(
        builder: (ctx, auth, ) => MaterialApp(
          title: 'SeeZen',
          theme: AppTheme.light,
          darkTheme: AppTheme.dark,
          themeMode: ThemeMode.system,
          localizationsDelegates: const [
            AppLocalizations.delegate,
            GlobalMaterialLocalizations.delegate,
            GlobalWidgetsLocalizations.delegate,
            GlobalCupertinoLocalizations.delegate,
          ],
          supportedLocales: const [
            Locale('fr'),
            Locale('en'),
          ],
          home: StreamBuilder<User?>(
            stream: FirebaseAuth.instance.authStateChanges(),
            builder: (context, snapshot) {
              if (snapshot.hasData) {
                return const HomeScreen();
              }
              return const AuthScreen();
            },
          ),
          routes: {
            HomeScreen.routeName: () => const HomeScreen(),
            ContactsScreen.routeName: () => const ContactsScreen(),
            WellbeingScreen.routeName: () => const WellbeingScreen(),
            SettingsScreen.routeName: (_) => const SettingsScreen(),
          },
        ),
      ),
    );
  }
}// lib/screens/home_screen.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

import 'contacts_screen.dart';
import 'wellbeing_screen.dart';
import 'settings_screen.dart';
import '../providers/auth_provider.dart';
import '../providers/plan_provider.dart';
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});
  static const routeName = '/home';
  @override
  Widget build(BuildContext context) {
    final loc = AppLocalizations.of(context)!;
    final plan = context.watch().plan;
return Scaffold(
  appBar: AppBar(
    title: const Text(
      'SeeZen',
      style: TextStyle(fontStyle: FontStyle.italic),
    ),
    actions: [
      IconButton(
        icon: const Icon(Icons.logout),
        onPressed: () => context.read<AuthProvider>().signOut(),
      ),
    ],
  ),
  body: Stack(
    children: [
      Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(loc.welcome, style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 20),
            ElevatedButton.icon(
              icon: const Icon(Icons.people),
              label: Text(loc.manageContacts),
              onPressed: () => Navigator.pushNamed(context, ContactsScreen.routeName),
            ),
            const SizedBox(height: 12),
            ElevatedButton.icon(
              icon: const Icon(Icons.spa),
              label: Text(loc.wellbeing),
              onPressed: () => Navigator.pushNamed(context, WellbeingScreen.routeName),
            ),
            const SizedBox(height: 12),
            ElevatedButton.icon(
              icon: const Icon(Icons.settings),
              label: Text(loc.settings),
              onPressed: () => Navigator.pushNamed(context, SettingsScreen.routeName),
            ),
            const SizedBox(height: 30),
            if (plan.type == PlanType.free)
              Text(loc.starterPlan, style: const TextStyle(color: Colors.grey))
            else
              Text('${loc.activePlan} ${plan.name}', style: const TextStyle(color: Colors.green)),
          ],
        ),
      ),
      // ⬇️ Mention en bas de l'écran
      const Align(
        alignment: Alignment.bottomCenter,
        child: Padding(
          padding: EdgeInsets.only(bottom: 16),
          child: Text(
            'Powered by SeeZen Labs®',
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey,
              fontWeight: FontWeight.w500,
            ),
          ),
        ),
      ),
    ],
  ),
);        
flutter pub get
flutter build apk --release --split-per-abi
{
  "@@locale": "en",
  "welcome": "Welcome to SeeZen",
  "manageContacts": "Manage contacts",
  "wellbeing": "Well-being",
  "settings": "Settings",
  "starterPlan": "Starter plan (free)",
  "activePlan": "Active plan:"
}
{
  "@@locale": "fr",
  "welcome": "Bienvenue dans SeeZen",
  "manageContacts": "Gérer les contacts",
  "wellbeing": "Bien-être",
  "settings": "Paramètres",
  "starterPlan": "Plan Starter (gratuit)",
  "activePlan": "Plan actif :"
}
Dans pubspec.yaml ajoute :

flutter:
  generate: true
Puis lance :

flutter gen-l10n
Crée un dossier seezen-backend :

src/main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(process.env.PORT || 3000);
}
bootstrap();
src/app.module.ts

import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { StripeModule } from './stripe/stripe.module';
import { UsersModule } from './users/users.module';
import { PlanModule } from './plan/plan.module';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: process.env.DB_HOST || 'localhost',
      port: +process.env.DB_PORT || 5432,
      username: process.env.DB_USER || 'postgres',
      password: process.env.DB_PASS || 'postgres',
      database: process.env.DB_NAME || 'seezen',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    UsersModule,
    PlanModule,
    StripeModule,
  ],
})
export class AppModule {}
src/stripe/stripe.service.ts

import { Injectable } from '@nestjs/common';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2023-10-16',
});
@Injectable()
export class StripeService {
  async createOneTimePayment(plan: 'pro' | 'premium') {
    const amount = plan === 'pro' ? 693 : 963; // cents
    const paymentIntent = await stripe.paymentIntents.create({
      amount,
      currency: 'usd',
      automatic_payment_methods: { enabled: true },
    });
    return { clientSecret: paymentIntent.client_secret };
  }
}
Dans lib/services/notification_service.dart :

import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/timezone.dart' as tz;

class NotificationService {
  final FlutterLocalNotificationsPlugin _local = FlutterLocalNotificationsPlugin();
  Future init() async {
    const AndroidInitializationSettings android =
        AndroidInitializationSettings('@mipmap/ic_launcher');
    final DarwinInitializationSettings iOS = DarwinInitializationSettings();
    final InitializationSettings settings =
        InitializationSettings(android: android, iOS: iOS);
    await _local.initialize(settings);
  }
  Future scheduleBirthdayReminder(
      int id, String name, DateTime birthday) async {
    final now = DateTime.now();
    final twoMonths = birthday.subtract(const Duration(days: 60));
    final threeWeeks = birthday.subtract(const Duration(days: 21));
    final threeDays = birthday.subtract(const Duration(days: 3));
    final sameDay = birthday;
await _schedule(id, '$name birthday in 2 months!', twoMonths);
await _schedule(id + 1, '$name birthday in 3 weeks!', threeWeeks);
await _schedule(id + 2, '$name birthday in 3 days!', threeDays);
await _schedule(id + 3, '$name birthday today!', sameDay);

  }
  Future _schedule(int id, String body, DateTime date) async {
    if (date.isBefore(DateTime.now())) return;
    await _local.zonedSchedule(
      id,
      'SeeZen Birthday Reminder',
      body,
      tz.TZDateTime.from(date, tz.local),
      const NotificationDetails(
        android: AndroidNotificationDetails('birthday', 'Birthday Reminders'),
        iOS: DarwinNotificationDetails(),
      ),
      uiLocalNotificationDateInterpretation:
          UILocalNotificationDateInterpretation.absoluteTime,
    );
  }
}


  }
}
